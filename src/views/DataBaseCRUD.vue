<template>
  <el-row>
    <el-col :offset="1" :span="9">
      <MetaDataTree :category.sync="category" :data-result.sync="tableData"></MetaDataTree>
    </el-col>
    <el-col :span="14">
      <DataCRUDTable :category.sync="category" :display-data.sync="tableData"></DataCRUDTable>
    </el-col>
  </el-row>
</template>
<script>
import MetaDataTree from '@/components/MetaDataTree.vue';
import DataCRUDTable from '@/components/DataCRUDTable.vue';

export default {
  name: 'DataBaseCRUD',
  components: {
    DataCRUDTable,
    MetaDataTree,
  },
  data() {
    return {
      tableData: [],
      category: [],
    };
  },
};
</script>

<style scoped>

</style>
