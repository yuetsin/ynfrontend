<template>
     <el-form label-position="right" label-width="auto">
        <el-form-item v-if="placeOrIndustry === 'place'" label="预测地区：">
          <el-select placeholder="请选择" v-model="postParams.region">
            <el-option
              v-for="item in predictRegions"
              :key="item"
              :value="item"
              :label="item">
            </el-option>
          </el-select>
        </el-form-item>
       <el-form-item v-if="placeOrIndustry === 'industry'" label="预测行业：">
         <el-select placeholder="请选择" v-model="postParams.industry">
           <el-option
             v-for="item in predictIndustries"
             :key="item"
             :value="item"
             :label="item">
           </el-option>
         </el-select>
       </el-form-item>
        <el-form-item v-if="!longTerm" label="预测方法：">
          <el-select placeholder="请选择" v-model="postParams.method">
            <el-option
              v-for="item in allMethods"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
      <el-form-item label="历史年份：">
        <year-range-selector
          :begin-year.sync='postParams.historyBeginYear'
          :end-year.sync="postParams.historyEndYear">
      </year-range-selector>
      </el-form-item>
       <el-form-item label="预测年份：">
         <year-range-selector
           :begin-year.sync='postParams.beginYear'
           :end-year.sync="postParams.endYear">
         </year-range-selector>
       </el-form-item>
      <el-form-item label="自变量：">
        <el-select placeholder="请选择" v-model="postParams.factor1.name">
          <el-option
            v-for="item in variadicFactors"
            :key="item"
            :label="item"
            :value="item">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="自变量是否有规划值：">
        <el-checkbox v-model="postParams.factor1.hasValue">
          {{ postParams.factor1.hasValue ? '有' : '无' }}
        </el-checkbox>
      </el-form-item>
       <el-form-item label="自变量规划值：" v-if="postParams.factor1.hasValue">
         <el-slider
           v-model="postParams.factor1.value"
           show-input
           :step="0.01"
           :max="1"
           :min="0">
         </el-slider>
       </el-form-item>

       <el-form-item label="第二自变量：" v-if="shouldDisplaySecondFactor">
         <el-select placeholder="请选择" v-model="postParams.factor2.name">
           <el-option
             v-for="item in variadicFactors"
             :key="item"
             :label="item"
             :value="item">
           </el-option>
         </el-select>
       </el-form-item>
       <el-form-item>
         <div v-if="shouldDisplaySecondFactor
         &&
         postParams.factor1.name.length > 0
         &&
         postParams.factor1.name === postParams.factor2.name"
         style="color: darkred; font-size: 12px">
           第二自变量不能和第一自变量相同。
         </div>
       </el-form-item>
       <el-form-item label="第二自变量是否有规划值：" v-if="shouldDisplaySecondFactor">
         <el-checkbox v-model="postParams.factor2.hasValue">
           {{ postParams.factor2.hasValue ? '有' : '无' }}
         </el-checkbox>
       </el-form-item>

       <el-form-item label="第二自变量规划值："
                     v-if="postParams.factor2.hasValue && shouldDisplaySecondFactor">
         <el-slider
           v-model="postParams.factor2.value"
           show-input
           :step="0.01"
           :max="1"
           :min="0">
         </el-slider>
       </el-form-item>
       <el-form-item label="方案标签：">
         <el-input clearable placeholder="可留空" v-model="postParams.tag">
         </el-input>
       </el-form-item>
       <el-form-item label="加载方案：">
         <el-select placeholder="选择标签" v-model="currentTag" size="small" style="width: 50%">
           <el-option
             v-for="item in knownTags"
             :key="item.id"
             :label="item.id"
             :value="item.id">
           </el-option>
         </el-select>
         <el-button size="small" @click="loadParameters"
                    :disabled="currentTag === null"
                    style="margin-left: 10px">加载</el-button>
       </el-form-item>
      <el-form-item>
        <el-button type="primary" :disabled="!canCommitQuery"
        @click="performPrediction">预测</el-button>
      </el-form-item>
     </el-form>

</template>

<script>
import { generateLabelAndValueObjsByArray } from '@/tool';
import YearRangeSelector from '@/components/YearRangeSelector.vue';

export default {
  name: 'SinglePredictSelectForm',
  components: { YearRangeSelector },
  data() {
    return {
      predictRegions: [],
      predictIndustries: [],
      variadicFactors: [],
      graphDataInternal: [],
      tableOneDataInternal: [],
      tableTwoDataInternal: [],
      postParams: {
        historyBeginYear: null,
        historyEndYear: null,
        beginYear: null,
        endYear: null,
        region: '',
        industry: '',
        method: '',
        factor1: {
          name: '',
          hasValue: true,
          value: 0.5,
        },
        factor2: {
          name: '',
          hasValue: true,
          value: 0.5,
        },
        tag: null,
        tagType: 'STATIC_REGIONAL',
      },
      originalAllMethodsForPlace: [],
      originalAllMethodsForIndustry: [],
      knownTags: [],
      currentTag: null,
    };
  },
  computed: {
    allMethods() {
      if (this.placeOrIndustry === 'place') {
        return generateLabelAndValueObjsByArray(this.originalAllMethodsForPlace);
      }
      if (this.placeOrIndustry === 'industry') {
        return generateLabelAndValueObjsByArray(this.originalAllMethodsForIndustry);
      }
      return [];
    },
    canCommitQuery() {
      const params = this.$data.postParams;
      if (params.beginYear === null || params.endYear === null) {
        return false;
      }
      if (params.historyBeginYear === null || params.historyEndYear === null) {
        return false;
      }
      if (params.region.length === 0) {
        return false;
      }
      if (!this.longTerm) {
        if (params.method.length === 0) {
          return false;
        }
      }
      if (!this.validateFactor(params.factor1)) {
        return false;
      }
      if (this.shouldDisplaySecondFactor) {
        if (params.factor1.name === params.factor2.name) {
          return false;
        }
        return this.validateFactor(params.factor2);
      }
      return true;
    },
    shouldDisplaySecondFactor() {
      return this.$data.postParams.method === '二元一次函数';
    },
  },
  mounted() {
    // this.loadParameters();
    if (this.placeOrIndustry === 'place') {
      this.loadRegions();
      this.loadRegionalMethods();
    } else {
      this.loadIndustries();
      this.loadIndustrialMethods();
    }
    this.loadFactors();
    this.loadTags();
  },
  methods: {
    loadTags() {
      this.$axios.get('/tags/query', {
        params: {
          tagType: 'STATIC_REGIONAL',
        },
      }).then((response) => {
        this.$data.knownTags = response.data.data;
      });
    },
    loadParameters() {
      this.$axios.get('/params/predict/static/region', {
        params: {
          tag: this.$data.currentTag,
        },
      }).then((response) => {
        this.$data.postParams = response.data.data;
      });
    },
    loadRegions() {
      this.$axios.get('/region/query').then((response) => {
        this.$data.predictRegions = response.data.data;
      });
    },
    loadIndustries() {
      this.$axios.get('/industry/query').then((response) => {
        this.$data.predictIndustries = response.data.data;
      });
    },
    loadFactors() {
      this.$axios.get('/factor/query').then((response) => {
        this.$data.variadicFactors = response.data.data;
      });
    },
    loadIndustrialMethods() {
      this.$axios.get('/method/industry/query').then((response) => {
        this.$data.originalAllMethodsForIndustry = response.data.data;
      });
    },
    loadRegionalMethods() {
      this.$axios.get('/method/region/query').then((response) => {
        this.$data.originalAllMethodsForPlace = response.data.data;
      });
    },
    validateFactor(factor) {
      return factor.name.length !== 0;
    },
    performPrediction() {
      this.$axios.post('/predict/region/single', this.$data.postParams).then((response) => {
        this.$data.graphDataInternal = response.data.data.graphData;
        this.$data.tableOneDataInternal = response.data.data.tableOneData;
        this.$data.tableTwoDataInternal = response.data.data.tableTwoData;
      });
    },
  },
  watch: {
    graphDataInternal(value) {
      this.$emit('update:graphData', value);
    },
    tableOneDataInternal(value) {
      this.$emit('update:tableOneData', value);
    },
    tableTwoDataInternal(value) {
      this.$emit('update:tableTwoData', value);
    },
  },
  props: [
    'placeOrIndustry',
    'longTerm',
    'graphData',
    'tableOneData',
    'tableTwoData',
  ],
};
</script>
<style scoped>
.el-input, .el-select {
  width: 60%
}
</style>
