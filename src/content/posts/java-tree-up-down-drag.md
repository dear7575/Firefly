---
title: ✨上移下移，升级降级树形节点操作工具类
published: 2025-01-23
pinned: false
description: "记录 Java 树形节点排序工具类实现，支持同层级上移下移、升级降级等节点调整操作。"
tags: [Java, 工具类]
category: 技术分享
draft: false
image: ../uploads/20260110/wl.jpg
---

# 上移下移，升级降级工具类

```java
package com.util;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.core.metadata.TableFieldInfo;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import com.baomidou.mybatisplus.core.metadata.TableInfoHelper;
import com.ActionResult;
import com.MsgCode;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.lang.reflect.Field;
import java.util.List;

/**
 * <strong style='color:purple;'>Created with IntelliJ IDEA.<hr>
 * <strong style='color:orange;'>Author: 北港不夏<hr>
 * <strong style='color:yellow;'>Date: 2025/1/20 16:07<hr>
 * <strong style='color:blue;'>Class: com.util<hr>
 * <strong style='color:indigo;'>Project: test<hr>
 * <strong style='color:red;'>Description: 上移下移、升级降级工具类<hr>
 */

@Slf4j
@Component
public class SequenceUtil {

    /**
     * 上移操作（仅限于同层级）
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param id            当前记录的ID
     * @param sequenceField 排序字段名
     * @param pidField      父级ID字段名
     * @param <T>           实体类型
     * @return {@link ActionResult}
     * @author 北港不夏
     * @date 2025/1/24 10:21
     */
    public <T> ActionResult moveUp(BaseMapper<T> mapper, Class<T> entityClass, String id, String sequenceField, String pidField, String categoryField) {
        // 获取当前记录
        T currentEntity = mapper.selectById(id);
        if (currentEntity == null) {
            return ActionResult.fail(MsgCode.FA002.get());
        }

        // 获取当前记录的 sequence 值、pid 值和分类 ID
        int currentSequence = getFieldValue(currentEntity, sequenceField, Integer.class);
        String currentPid = getFieldValue(currentEntity, pidField, String.class);
        String categoryId = getFieldValue(currentEntity, categoryField, String.class);

        // 构建查询条件：找到同层级的前一个记录
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        if (currentPid != null) {
            // 同层级
            queryWrapper.eq(pidField, currentPid); 
        } else {
            // 顶级节点
            queryWrapper.isNull(pidField); 
        }
        // 分类 ID
        queryWrapper.eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // sequence 小于当前值
                .lt(sequenceField, currentSequence)
                // 按 sequence 降序
                .orderByDesc(sequenceField)
                // 只取一条
                .last("LIMIT 1");
        T previousEntity = mapper.selectOne(queryWrapper);

        if (previousEntity != null) {
            // 获取前一个记录的 sequence 值
            int previousSequence = getFieldValue(previousEntity, sequenceField, Integer.class);

            // 交换 sequence 值
            setFieldValue(currentEntity, sequenceField, previousSequence);
            setFieldValue(previousEntity, sequenceField, currentSequence);

            // 更新数据库
            mapper.updateById(currentEntity);
            mapper.updateById(previousEntity);

            // 重新排序同层级的所有记录
            reorderSequence(mapper, entityClass, sequenceField, pidField, categoryField, currentPid, categoryId);
        }
        return ActionResult.success(MsgCode.SU004.get());
    }

    /**
     * 下移操作（仅限于同层级）
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param id            当前记录的ID
     * @param sequenceField 排序字段名
     * @param pidField      父级ID字段名
     * @param <T>           实体类型
     * @return {@link ActionResult}
     * @author 北港不夏
     * @date 2025/1/24 10:21
     */
    public <T> ActionResult moveDown(BaseMapper<T> mapper, Class<T> entityClass, String id, String sequenceField, String pidField, String categoryField) {
        // 获取当前记录
        T currentEntity = mapper.selectById(id);
        if (currentEntity == null) {
            return ActionResult.fail(MsgCode.FA002.get());
        }

        // 获取当前记录的 sequence 值、pid 值和分类 ID
        int currentSequence = getFieldValue(currentEntity, sequenceField, Integer.class);
        String currentPid = getFieldValue(currentEntity, pidField, String.class);
        String categoryId = getFieldValue(currentEntity, categoryField, String.class);

        // 构建查询条件：找到同层级的后一个记录
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        if (currentPid != null) {
            // 同层级
            queryWrapper.eq(pidField, currentPid);
        } else {
            // 顶级节点
            queryWrapper.isNull(pidField);
        }
        // 分类 ID
        queryWrapper.eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // sequence 大于当前值
                .gt(sequenceField, currentSequence)
                // 按 sequence 升序
                .orderByAsc(sequenceField)
                // 只取一条
                .last("LIMIT 1");
        T nextEntity = mapper.selectOne(queryWrapper);

        if (nextEntity != null) {
            // 获取后一个记录的 sequence 值
            int nextSequence = getFieldValue(nextEntity, sequenceField, Integer.class);

            // 交换 sequence 值
            setFieldValue(currentEntity, sequenceField, nextSequence);
            setFieldValue(nextEntity, sequenceField, currentSequence);

            // 更新数据库
            mapper.updateById(currentEntity);
            mapper.updateById(nextEntity);

            // 重新排序同层级的所有记录
            reorderSequence(mapper, entityClass, sequenceField, pidField, categoryField, currentPid, categoryId);
        }
        return ActionResult.success(MsgCode.SU004.get());
    }

    /**
     * 重新排序同层级的所有记录，确保 sequence 从 1 开始连续递增
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param sequenceField 排序字段名
     * @param pidField      父级ID字段名
     * @param categoryField 分类字段名
     * @param pid           父级ID
     * @param categoryId    分类ID
     * @param <T>           实体类型
     */
    private <T> void reorderSequence(BaseMapper<T> mapper, Class<T> entityClass, String sequenceField, String pidField, String categoryField, String pid, String categoryId) {
        // 构建查询条件：获取同层级的所有记录
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        if (pid != null) {
            queryWrapper.eq(pidField, pid); // 同层级
        } else {
            queryWrapper.isNull(pidField); // 顶级节点
        }
        // 分类 ID
        queryWrapper.eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // 按 sequence 升序
                .orderByAsc(sequenceField);

        // 查询同层级的所有记录
        List<T> entityList = mapper.selectList(queryWrapper);

        // 重新设置 sequence 值，从 1 开始连续递增
        int sequence = 1;
        for (T entity : entityList) {
            setFieldValue(entity, sequenceField, sequence++);
            mapper.updateById(entity);
        }
    }

    /**
     * 升级操作（跨层级）
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param id            当前记录的ID
     * @param sequenceField 排序字段名
     * @param pidField      父级ID字段名
     * @param idField       主键字段名
     * @param <T>           实体类型
     * @return {@link ActionResult}
     * @author 北港不夏
     * @date 2025/1/24 10:21
     */
    public <T> ActionResult moveLevelUp(BaseMapper<T> mapper, Class<T> entityClass, String id, String sequenceField, String pidField, String idField, String categoryField) {
        // 获取当前记录
        T currentEntity = mapper.selectById(id);
        if (currentEntity == null) {
            return ActionResult.fail(MsgCode.FA002.get());
        }

        // 获取当前记录的 pid 值
        String currentPid = getFieldValue(currentEntity, pidField, String.class);
        if (currentPid == null) {
            return ActionResult.fail(MsgCode.FA002.get() + "，当前节点已是最顶级节点");
        }

        // 找到父级记录
        T parentEntity = mapper.selectById(currentPid);
        if (parentEntity == null) {
            return ActionResult.fail(MsgCode.FA002.get() + "，未找到父级节点");
        }

        // 获取父级记录的 pid 值（即祖父级 pid）
        String grandParentPid = getFieldValue(parentEntity, pidField, String.class);

        // 获取父级记录的 sequence 值
        Integer parentSequence = getFieldValue(parentEntity, sequenceField, Integer.class);
        if (parentSequence == null) {
            return ActionResult.fail(MsgCode.FA002.get() + "，父级节点 sequence 值为空");
        }

        // 获取拖拽节点的分类ID（使用实体类属性名）
        String categoryId = getFieldValue(currentEntity, categoryField, String.class);

        // 将当前记录的 pid 设置为父级记录的 pid（即升级到祖父级）
        setFieldValue(currentEntity, pidField, grandParentPid);

        // 将当前记录的 sequence 设置为父级记录的 sequence + 1
        setFieldValue(currentEntity, sequenceField, parentSequence + 1);

        // 使用 UpdateWrapper 更新当前记录的 pid 和 sequence 值
        UpdateWrapper<T> updateWrapper = new UpdateWrapper<>();
        // 条件：id = 当前记录ID
        updateWrapper.eq(idField, id)
                // 分类 ID
                .eq(getDatabaseColumnName(entityClass, categoryField), categoryId);
        // 设置为父级记录的 pid
        updateWrapper.set(pidField, grandParentPid)
                // 设置为父级记录的 sequence + 1
                .set(sequenceField, parentSequence + 1);
        mapper.update(null, updateWrapper);

        // 重新刷新目标层级的所有节点的 sequence 值，从 1 开始
        QueryWrapper<T> targetQueryWrapper = new QueryWrapper<>();
        if (grandParentPid != null) {
            // 同层级
            targetQueryWrapper.eq(pidField, grandParentPid);
        } else {
            // 顶级节点
            targetQueryWrapper.isNull(pidField);
        }
        // 分类 ID
        targetQueryWrapper.eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // 按 sequence 升序
                .orderByAsc(sequenceField);
        List<T> targetEntities = mapper.selectList(targetQueryWrapper);

        int sequence = 1;
        for (T entity : targetEntities) {
            setFieldValue(entity, sequenceField, sequence++);
            mapper.updateById(entity);
        }

        // 重新刷新原同级节点的 sequence 值，从 1 开始
        QueryWrapper<T> siblingQueryWrapper = new QueryWrapper<>();
        // 条件：pid = 当前记录的父级ID
        siblingQueryWrapper.eq(pidField, currentPid)
                // 分类 ID
                .eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                .orderByAsc(sequenceField); // 按 sequence 升序
        List<T> siblingEntities = mapper.selectList(siblingQueryWrapper);

        sequence = 1;
        for (T entity : siblingEntities) {
            setFieldValue(entity, sequenceField, sequence++);
            mapper.updateById(entity);
        }

        // 获取当前节点的子节点（如果有）
        QueryWrapper<T> childQueryWrapper = new QueryWrapper<>();
        // 条件：pid = 当前节点ID
        childQueryWrapper.eq(pidField, id)
                // 分类 ID
                .eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // 按 sequence 升序
                .orderByAsc(sequenceField);
        List<T> childEntities = mapper.selectList(childQueryWrapper);

        // 重新计算子节点的 sequence 值，从 1 开始
        sequence = 1;
        for (T childEntity : childEntities) {
            setFieldValue(childEntity, sequenceField, sequence++);
            mapper.updateById(childEntity);
        }

        return ActionResult.success(MsgCode.SU004.get());
    }

    /**
     * 降级操作（跨层级）
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param id            当前记录的ID
     * @param sequenceField 排序字段名
     * @param pidField      父级ID字段名
     * @param idField       主键字段名
     * @param <T>           实体类型
     * @return {@link ActionResult}
     * @author 北港不夏
     * @date 2025/1/24 10:20
     */
    public <T> ActionResult moveLevelDown(
            BaseMapper<T> mapper, Class<T> entityClass,
            String id, String sequenceField, String pidField, String idField,
            String categoryField) {
        // 获取当前记录
        T currentEntity = mapper.selectById(id);
        if (currentEntity == null) {
            return ActionResult.fail(MsgCode.FA002.get());
        }

        // 获取当前记录的 pid 值和 sequence 值
        String currentPid = getFieldValue(currentEntity, pidField, String.class);
        int currentSequence = getFieldValue(currentEntity, sequenceField, Integer.class);

        // 获取拖拽节点的分类ID（使用实体类属性名）
        String categoryId = getFieldValue(currentEntity, categoryField, String.class);

        // 找到前一个兄弟节点
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        if (currentPid != null) {
            // 同层级
            queryWrapper.eq(pidField, currentPid);
        } else {
            // 顶级节点
            queryWrapper.isNull(pidField);
        }
        // 分类 ID
        queryWrapper.eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // sequence 小于当前值
                .lt(sequenceField, currentSequence)
                // 按 sequence 降序
                .orderByDesc(sequenceField)
                // 只取一条
                .last("LIMIT 1");
        T previousEntity = mapper.selectOne(queryWrapper);

        if (previousEntity == null) {
            return ActionResult.fail(MsgCode.FA102.get() + "，未找到同级前面节点");
        }

        // 将当前记录的 pid 设置为前一个兄弟节点的 id
        String previousId = getFieldValue(previousEntity, idField, String.class);
        setFieldValue(currentEntity, pidField, previousId);

        // 更新当前记录
        UpdateWrapper<T> updateWrapper = new UpdateWrapper<>();
        // 条件：id = 当前记录ID
        updateWrapper.eq(idField, id)
                // 分类 ID
                .eq(getDatabaseColumnName(entityClass, categoryField), categoryId);
        // 设置为前一个兄弟节点的 id
        updateWrapper.set(pidField, previousId);
        mapper.update(null, updateWrapper);

        // 重新刷新当前记录的同级节点的 sequence（即原来的同级节点）
        QueryWrapper<T> siblingQueryWrapper = new QueryWrapper<>();
        if (currentPid != null) {
            // 同层级
            siblingQueryWrapper.eq(pidField, currentPid);
        } else {
            // 顶级节点
            siblingQueryWrapper.isNull(pidField);
        }
        // 分类 ID
        siblingQueryWrapper.eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // 按 sequence 升序
                .orderByAsc(sequenceField);
        List<T> siblingEntities = mapper.selectList(siblingQueryWrapper);

        // 重新计算同级节点的 sequence 值，从 1 开始
        int sequence = 1;
        for (T entity : siblingEntities) {
            setFieldValue(entity, sequenceField, sequence++);
            mapper.updateById(entity);
        }

        // 将当前记录移动到子级的最后
        QueryWrapper<T> childQueryWrapper = new QueryWrapper<>();
        // 条件：pid = 前一个兄弟节点的 id
        childQueryWrapper.eq(pidField, previousId)
                // 分类 ID
                .eq(getDatabaseColumnName(entityClass, categoryField), categoryId)
                // 排除当前记录本身
                .ne(idField, id)
                // 按 sequence 降序
                .orderByDesc(sequenceField);
        List<T> childEntities = mapper.selectList(childQueryWrapper);

        // 找到子节点中最大的 sequence 值
        int maxChildSequence = 0;
        if (!childEntities.isEmpty()) {
            // 子节点中 sequence 最大的记录
            T lastChildEntity = childEntities.getFirst();
            maxChildSequence = getFieldValue(lastChildEntity, sequenceField, Integer.class);
        }

        // 将当前记录的 sequence 设置为子节点中最大的 sequence + 1
        setFieldValue(currentEntity, sequenceField, maxChildSequence + 1);
        mapper.updateById(currentEntity);

        return ActionResult.success(MsgCode.SU004.get());
    }

    /**
     * 拖拽操作
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param dragId        拖拽的节点ID
     * @param targetId      目标节点ID
     * @param dragType      拖拽类型（before, after, inner）
     * @param sequenceField 排序字段名（实体类属性名）
     * @param pidField      父级ID字段名（实体类属性名）
     * @param idField       主键字段名（实体类属性名）
     * @param categoryField 分类ID字段名（实体类属性名）
     * @param <T>           实体类型
     * @author 北港不夏
     * @date 2025/1/23 14:19
     */
    public <T> void drag(BaseMapper<T> mapper, Class<T> entityClass, String dragId, String targetId, String dragType, String sequenceField, String pidField, String idField, String categoryField) {
        // 获取拖拽节点和目标节点
        T dragEntity = mapper.selectById(dragId);
        T targetEntity = mapper.selectById(targetId);
        if (dragEntity == null || targetEntity == null) {
            throw new RuntimeException("Drag or target record not found");
        }

        // 获取拖拽节点的分类ID（使用实体类属性名）
        String categoryId = getFieldValue(dragEntity, categoryField, String.class);

        // 获取目标节点的父级ID（使用实体类属性名）
        String targetPid = getFieldValue(targetEntity, pidField, String.class);

        // 根据拖拽类型处理
        switch (dragType) {
            case "before":
            case "after":
                // 拖拽到目标节点的前面或后面
                handleDragBeforeOrAfter(mapper, entityClass, dragEntity, targetEntity, dragType, sequenceField, pidField, idField, categoryField, categoryId, targetPid);
                break;
            case "inner":
                // 拖拽到目标节点内部（成为其子节点）
                handleDragInner(mapper, entityClass, dragEntity, targetEntity, null, null, sequenceField, pidField, idField, categoryField, categoryId);
                break;
            case "outer":
                // 拖拽到目标节点外部（与目标节点同层级）
                handleDragOuter(mapper, entityClass, dragEntity, targetEntity, sequenceField, pidField, idField, categoryField, categoryId);
                break;
            default:
                throw new RuntimeException("Invalid drag type: " + dragType);
        }
    }

    /**
     * 处理拖拽到目标节点前面或后面的逻辑
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param dragEntity    拖拽的对象
     * @param targetEntity  目标的对象
     * @param dragType      拖拽类型（before, after, inner）
     * @param sequenceField 排序字段名（实体类属性名）
     * @param pidField      父级ID字段名（实体类属性名）
     * @param idField       主键字段名（实体类属性名）
     * @param categoryField 分类ID字段名（实体类属性名）
     * @param categoryId    拖拽节点的分类ID（使用实体类属性名）
     * @param targetPid     目标节点的父级ID（使用实体类属性名）
     * @param <T>           实体类型
     * @author 北港不夏
     * @date 2025/1/23 14:19
     */
    private <T> void handleDragBeforeOrAfter(BaseMapper<T> mapper, Class<T> entityClass, T dragEntity, T targetEntity, String dragType, String sequenceField, String pidField, String idField, String categoryField, String categoryId, String targetPid) {
        // 获取分类ID的数据库列名
        String dbCategoryColumn = getDatabaseColumnName(entityClass, categoryField);

        // 获取拖拽节点和目标节点的 sequence 值（使用实体类属性名）
        int dragSequence = getFieldValue(dragEntity, sequenceField, Integer.class);
        int targetSequence = getFieldValue(targetEntity, sequenceField, Integer.class);

        // 更新拖拽节点的 pid
        UpdateWrapper<T> updateWrapper = new UpdateWrapper<>();
        // 使用实体类属性名
        updateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
        // 使用数据库列名
        updateWrapper.eq(dbCategoryColumn, categoryId);
        // 使用实体类属性名
        updateWrapper.set(pidField, targetPid);
        mapper.update(null, updateWrapper);

        // 获取同层级的所有节点（包括拖拽节点）
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        if (targetPid != null) {
            // 使用实体类属性名
            queryWrapper.eq(pidField, targetPid);
        } else {
            // 使用实体类属性名
            queryWrapper.isNull(pidField);
        }
        // 使用数据库列名
        queryWrapper.eq(dbCategoryColumn, categoryId);
        // 使用实体类属性名
        queryWrapper.orderByAsc(sequenceField);
        List<T> siblingEntities = mapper.selectList(queryWrapper);

        // 重新计算 sequence 值
        int newSequence = (dragType.equals("before")) ? targetSequence != 1 ? dragSequence - 1 : targetSequence : targetSequence;

        // 重新排序同层级节点的 sequence 值
        int sequence = 1;
        for (T entity : siblingEntities) {
            String entityId = getFieldValue(entity, idField, String.class);
            if (entityId.equals(getFieldValue(dragEntity, idField, String.class))) {
                // 跳过拖拽节点
                continue;
            }
            if (sequence == newSequence) {
                // 为拖拽节点留出位置
                sequence++;
            }
            setFieldValue(entity, sequenceField, sequence++);
            mapper.updateById(entity);
        }

        // 更新拖拽节点的 sequence
        setFieldValue(dragEntity, sequenceField, newSequence);
        mapper.updateById(dragEntity);
    }

    /**
     * 处理拖拽到目标节点内部的逻辑
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param dragEntity    拖拽的对象
     * @param targetEntity  目标的对象
     * @param dragType      拖拽类型（before, after, inner）
     * @param targetChildId 目标子节点的 ID
     * @param sequenceField 排序字段名（实体类属性名）
     * @param pidField      父级ID字段名（实体类属性名）
     * @param idField       主键字段名（实体类属性名）
     * @param categoryField 分类ID字段名（实体类属性名）
     * @param categoryId    拖拽节点的分类ID值
     * @param <T>           实体类型
     * @author 北港不夏
     * @date 2025/1/23 14:19
     */
    private <T> void handleDragInner(BaseMapper<T> mapper, Class<T> entityClass, T dragEntity, T targetEntity, String dragType, String targetChildId, String sequenceField, String pidField, String idField, String categoryField, String categoryId) {
        // 获取分类ID的数据库列名
        String dbCategoryColumn = getDatabaseColumnName(entityClass, categoryField);

        // 获取目标节点的 ID（使用实体类属性名）
        String targetId = getFieldValue(targetEntity, idField, String.class);

        // 更新拖拽节点的 pid
        UpdateWrapper<T> updateWrapper = new UpdateWrapper<>();
        // 使用实体类属性名
        updateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
        // 使用数据库列名
        updateWrapper.eq(dbCategoryColumn, categoryId);
        // 使用目标节点的 ID
        updateWrapper.set(pidField, targetId);
        mapper.update(null, updateWrapper);

        // 获取目标节点的子节点
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        // 使用实体类属性名
        queryWrapper.eq(pidField, targetId)
                // 使用数据库列名
                .eq(dbCategoryColumn, categoryId)
                // 使用实体类属性名
                .orderByAsc(sequenceField);
        List<T> childEntities = mapper.selectList(queryWrapper);

        // 重新计算目标节点子节点的 sequence 值
        int newSequence = 1;
        // 标记拖拽节点是否已处理
        boolean isDragNodeProcessed = false;
        for (T entity : childEntities) {
            // 如果当前节点是拖拽节点，则跳过，稍后处理
            if (getFieldValue(entity, idField, String.class).equals(getFieldValue(dragEntity, idField, String.class))) {
                continue;
            }

            // 如果指定了 targetChildId，并且当前节点是目标子节点
            if (targetChildId != null &&
                    getFieldValue(entity, idField, String.class).equals(targetChildId)) {
                // 如果是 before，则将拖拽节点插入到目标子节点之前
                if ("before".equals(dragType)) {
                    UpdateWrapper<T> dragSequenceUpdateWrapper = new UpdateWrapper<>();
                    // 使用实体类属性名
                    dragSequenceUpdateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
                    // 使用数据库列名
                    dragSequenceUpdateWrapper.eq(dbCategoryColumn, categoryId);
                    // 设置 sequence
                    dragSequenceUpdateWrapper.set(sequenceField, newSequence++);
                    mapper.update(null, dragSequenceUpdateWrapper);
                    isDragNodeProcessed = true;
                }

                // 更新目标子节点的 sequence
                UpdateWrapper<T> targetSequenceUpdateWrapper = new UpdateWrapper<>();
                // 使用实体类属性名
                targetSequenceUpdateWrapper.eq(idField, getFieldValue(entity, idField, String.class));
                // 使用数据库列名
                targetSequenceUpdateWrapper.eq(dbCategoryColumn, categoryId);
                // 设置 sequence
                targetSequenceUpdateWrapper.set(sequenceField, newSequence++);
                mapper.update(null, targetSequenceUpdateWrapper);

                // 如果是 after，则将拖拽节点插入到目标子节点之后
                if ("after".equals(dragType)) {
                    UpdateWrapper<T> dragSequenceUpdateWrapper = new UpdateWrapper<>();
                    // 使用实体类属性名
                    dragSequenceUpdateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
                    // 使用数据库列名
                    dragSequenceUpdateWrapper.eq(dbCategoryColumn, categoryId);
                    // 设置 sequence
                    dragSequenceUpdateWrapper.set(sequenceField, newSequence++);
                    mapper.update(null, dragSequenceUpdateWrapper);
                    isDragNodeProcessed = true;
                }
            } else {
                // 其他节点按顺序设置 sequence
                UpdateWrapper<T> sequenceUpdateWrapper = new UpdateWrapper<>();
                // 使用实体类属性名
                sequenceUpdateWrapper.eq(idField, getFieldValue(entity, idField, String.class));
                // 使用数据库列名
                sequenceUpdateWrapper.eq(dbCategoryColumn, categoryId);
                // 设置 sequence
                sequenceUpdateWrapper.set(sequenceField, newSequence++);
                mapper.update(null, sequenceUpdateWrapper);
            }
        }

        // 如果拖拽节点未被处理（例如目标节点是最后一个节点或未指定 targetChildId），则将其 sequence 设置为最大值
        if (!isDragNodeProcessed) {
            UpdateWrapper<T> dragSequenceUpdateWrapper = new UpdateWrapper<>();
            // 使用实体类属性名
            dragSequenceUpdateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
            // 使用数据库列名
            dragSequenceUpdateWrapper.eq(dbCategoryColumn, categoryId);
            // 设置 sequence
            dragSequenceUpdateWrapper.set(sequenceField, newSequence);
            mapper.update(null, dragSequenceUpdateWrapper);
        }

        // 重新排序外部节点的 sequence 值
        reorderSiblingSequences(mapper, entityClass, dragEntity, sequenceField, pidField, idField, categoryField, categoryId);
    }

    /**
     * 重新排序外部节点的 sequence 值
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param dragEntity    拖拽的对象
     * @param sequenceField 排序字段名（实体类属性名）
     * @param pidField      父级ID字段名（实体类属性名）
     * @param idField       主键字段名（实体类属性名）
     * @param categoryField 分类ID字段名（实体类属性名）
     * @param categoryId    拖拽节点的分类ID值
     * @param <T>           实体类型
     * @author 北港不夏
     * @date 2025/1/23 14:23
     */
    private <T> void reorderSiblingSequences(BaseMapper<T> mapper, Class<T> entityClass, T dragEntity, String sequenceField, String pidField, String idField, String categoryField, String categoryId) {
        // 获取分类ID的数据库列名
        String dbCategoryColumn = getDatabaseColumnName(entityClass, categoryField);

        // 获取拖拽节点的父级ID（使用实体类属性名）
        String dragPid = getFieldValue(dragEntity, pidField, String.class);

        // 获取同层级的所有节点（不包括拖拽节点）
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        if (dragPid != null) {
            // 使用实体类属性名
            queryWrapper.eq(pidField, dragPid);
        } else {
            // 使用实体类属性名
            queryWrapper.isNull(pidField);
        }
        // 使用数据库列名
        queryWrapper.eq(dbCategoryColumn, categoryId);
        // 排除拖拽节点
        queryWrapper.ne(idField, getFieldValue(dragEntity, idField, String.class));
        // 使用实体类属性名
        queryWrapper.orderByAsc(sequenceField);
        List<T> siblingEntities = mapper.selectList(queryWrapper);

        // 重新计算外部节点的 sequence 值
        int sequence = 1;
        for (T entity : siblingEntities) {
            setFieldValue(entity, sequenceField, sequence++);
            mapper.updateById(entity);
        }
    }

    /**
     * 处理拖拽到目标节点外部的逻辑
     *
     * @param mapper        MyBatis-Plus 的 Mapper 接口
     * @param entityClass   实体类的 Class 对象
     * @param dragEntity    拖拽的对象
     * @param targetEntity  目标的对象
     * @param sequenceField 排序字段名（实体类属性名）
     * @param pidField      父级ID字段名（实体类属性名）
     * @param idField       主键字段名（实体类属性名）
     * @param categoryField 分类ID字段名（实体类属性名）
     * @param categoryId    拖拽节点的分类ID值
     * @param <T>           实体类型
     * @author 北港不夏
     * @date 2025/1/23 14:24
     */
    private <T> void handleDragOuter(BaseMapper<T> mapper, Class<T> entityClass, T dragEntity, T targetEntity, String sequenceField, String pidField, String idField, String categoryField, String categoryId) {
        // 获取分类ID的数据库列名
        String dbCategoryColumn = getDatabaseColumnName(entityClass, categoryField);

        // 获取目标节点的父级ID（使用实体类属性名）
        String targetPid = getFieldValue(targetEntity, pidField, String.class);

        // 更新拖拽节点的 pid（与目标节点同层级）
        UpdateWrapper<T> updateWrapper = new UpdateWrapper<>();
        // 使用实体类属性名
        updateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
        // 使用数据库列名
        updateWrapper.eq(dbCategoryColumn, categoryId);
        // 使用目标节点的 pid
        updateWrapper.set(pidField, targetPid);
        mapper.update(null, updateWrapper);

        // 获取目标节点的同层级节点（包括拖拽节点）
        QueryWrapper<T> queryWrapper = new QueryWrapper<>();
        if (targetPid != null) {
            // 使用实体类属性名
            queryWrapper.eq(pidField, targetPid);
        } else {
            // 使用实体类属性名
            queryWrapper.isNull(pidField);
        }
        // 使用数据库列名
        queryWrapper.eq(dbCategoryColumn, categoryId);
        // 使用实体类属性名
        queryWrapper.orderByAsc(sequenceField);
        List<T> siblingEntities = mapper.selectList(queryWrapper);

        // 重新计算同层级节点的 sequence 值
        int sequence = 1;
        // 标记拖拽节点是否已处理
        boolean isDragNodeProcessed = false;
        for (T entity : siblingEntities) {
            // 如果当前节点是拖拽节点，则跳过，稍后处理
            if (getFieldValue(entity, idField, String.class).equals(getFieldValue(dragEntity, idField, String.class))) {
                continue;
            }

            // 如果当前节点是目标节点，则将拖拽节点插入到目标节点之后
            if (getFieldValue(entity, idField, String.class).equals(getFieldValue(targetEntity, idField, String.class))) {
                // 设置目标节点的 sequence
                setFieldValue(entity, sequenceField, sequence++);
                mapper.updateById(entity);

                // 设置拖拽节点的 sequence 为目标节点的 sequence + 1
                UpdateWrapper<T> sequenceUpdateWrapper = new UpdateWrapper<>();
                // 使用实体类属性名
                sequenceUpdateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
                // 使用数据库列名
                sequenceUpdateWrapper.eq(dbCategoryColumn, categoryId);
                // 设置 sequence
                sequenceUpdateWrapper.set(sequenceField, sequence++);
                mapper.update(null, sequenceUpdateWrapper);
                isDragNodeProcessed = true;
            } else {
                // 其他节点按顺序设置 sequence
                setFieldValue(entity, sequenceField, sequence++);
                mapper.updateById(entity);
            }
        }

        // 如果拖拽节点未被处理（例如目标节点是最后一个节点），则将其 sequence 设置为最大值
        if (!isDragNodeProcessed) {
            UpdateWrapper<T> sequenceUpdateWrapper = new UpdateWrapper<>();
            // 使用实体类属性名
            sequenceUpdateWrapper.eq(idField, getFieldValue(dragEntity, idField, String.class));
            // 使用数据库列名
            sequenceUpdateWrapper.eq(dbCategoryColumn, categoryId);
            // 设置 sequence
            sequenceUpdateWrapper.set(sequenceField, sequence);
            mapper.update(null, sequenceUpdateWrapper);
        }
    }

    /**
     * 通过反射设置字段的值
     *
     * @param entity    实体对象
     * @param fieldName 实体类属性名
     * @param value     字段值
     * @param <T>       实体类型
     * @author 北港不夏
     * @date 2025/1/23 14:24
     */
    private <T> void setFieldValue(T entity, String fieldName, Object value) {
        try {
            Field field = entity.getClass().getDeclaredField(fieldName);
            field.setAccessible(true);
            field.set(entity, value);
        } catch (Exception e) {
            throw new RuntimeException("Failed to set field value: " + fieldName, e);
        }
    }

    /**
     * 通过反射获取字段的值
     *
     * @param entity    实体对象
     * @param fieldName 实体类属性名
     * @param fieldType 字段类型
     * @param <T>       实体类型
     * @param <R>       字段类型
     * @return 字段值
     * @author 北港不夏
     * @date 2025/1/23 14:19
     */
    private <T, R> R getFieldValue(T entity, String fieldName, Class<R> fieldType) {
        try {
            Field field = entity.getClass().getDeclaredField(fieldName);
            field.setAccessible(true);
            return fieldType.cast(field.get(entity));
        } catch (Exception e) {
            throw new RuntimeException("Failed to get field value: " + fieldName, e);
        }
    }

    /**
     * 获取字段的数据库列名
     *
     * @param entityClass 实体类的 Class 对象
     * @param fieldName   实体类属性名
     * @param <T>         实体类型
     * @return 数据库列名
     * @author 北港不夏
     * @date 2025/1/23 14:19
     */
    private <T> String getDatabaseColumnName(Class<T> entityClass, String fieldName) {
        TableInfo tableInfo = TableInfoHelper.getTableInfo(entityClass);
        if (tableInfo != null) {
            return tableInfo.getFieldList().stream()
                    // 匹配属性名
                    .filter(field -> field.getProperty().equals(fieldName))
                    .findFirst()
                    // 获取数据库列名
                    .map(TableFieldInfo::getColumn)
                    // 如果没有找到，则返回字段名
                    .orElse(fieldName);
        }
        return fieldName;
    }
}

```

