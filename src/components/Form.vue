<script>
  import { bitable, FieldType } from '@lark-base-open/js-sdk';
  import { ref, watch, onMounted, reactive } from 'vue';
  import {
    ElButton,
    ElForm,
    ElFormItem,
    ElSelect,
    ElOption,
  } from 'element-plus';

  export default {
    components: {
      ElButton,
      ElForm,
      ElFormItem,
      ElSelect,
      ElOption,
    },
    setup() {
      const formData = reactive({ table: '', field: '' });
      const tableMetaList = ref([]);
      const fieldMetaList = ref([]);

      onMounted(async () => {
        const [tableList, selection] = await Promise.all([bitable.base.getTableMetaList(), bitable.base.getSelection()]);
        formData.table = selection.tableId;
        tableMetaList.value = tableList;
      });

      watch(() => formData.table, async (newVal, oldVal) => {
        if (newVal !== oldVal) {
          const table = await bitable.base.getTableById(newVal);
          fieldMetaList.value = await table.getFieldMetaList();
        }
      });

      const changeTimeStamp = async () => {
        if (formData.table && formData.field) {
          const table = await bitable.base.getTableById(formData.table);
          const recordIdList = await table.getRecordIdList();
          const field = await table.getFieldById(formData.field);
          const fieldName = await field.getName()
          const dateField = await table.addField({
            type: FieldType.Text,
            name: fieldName + '_transfer'
          })
          for (var i = 0; i < recordIdList.length; i++) {
            var timestamp = await table.getCellValue(formData.field, recordIdList[i]);
            timestamp = timestamp[0];
            const data = new Date(Number(timestamp.text)).toLocaleString();
            const res = table.setCellValue(dateField, recordIdList[i], [
              {
                type: 'text',
                text: data
              }
            ]);
          }
        }
      };

      return {
        formData,
        tableMetaList,
        fieldMetaList,
        changeTimeStamp,
      };
    },
  };
</script>

<template>
  <el-form ref="form" class="form" :model="formData" label-position="top">
    <el-form-item label="选择数据表" size="large">
        <el-select v-model="formData.table" placeholder="请选择数据表" style="width: 100%">
          <el-option
            v-for="meta in tableMetaList"
            :key="meta.id"
            :label="meta.name"
            :value="meta.id"
          />
        </el-select>
    </el-form-item>
    <el-form-item label="选择时间戳字段" size="large">
        <el-select v-model="formData.field" placeholder="请选择时间戳字段" style="width: 100%">
          <el-option
            v-for="meta in fieldMetaList"
            :key="meta.id"
            :label="meta.name"
            :value="meta.id"
          />
        </el-select>
    </el-form-item>
    <el-button type="primary" plain size="large" @click="changeTimeStamp">转换时间戳</el-button>
  </el-form>
</template>

<style scoped>
  .form :deep(.el-form-item__label) {
    font-size: 16px;
    color: var(--el-text-color-primary);
    margin-bottom: 0;
  }
  .form :deep(.el-form-item__content) {
    font-size: 16px;
  }
</style>
