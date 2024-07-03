<template>
  <div v-if="!dataSource.length" class="form">
    <h3>电费计算</h3>
    <a-form :model="formData" :label-col="{ span:6 }" :wrapper-col="{ flex: 12 }" >
      <a-form-item label="电费充值金额">
        <a-input v-model:value="formData.money"  placeholder="0.00" style="width: 130px" />
      </a-form-item>
      <a-form-item label="房间数">
        <a-input v-model:value="formData.number" style="width: 130px" />
      </a-form-item>
      <a-form-item label="电表3" >
        <a-input v-model:value="formData.number_3.current" placeholder="当前记录"  style="width: 130px" />
        <span>-</span>
        <a-input v-model:value="formData.number_3.last" placeholder="上次记录" style="width: 130px" />
      </a-form-item>
      <a-form-item label="电表1">
        <a-input v-model:value="formData.number_1.current" placeholder="当前记录" style="width: 130px" />
        <span>-</span>
        <a-input v-model:value="formData.number_1.last" placeholder="上次记录" style="width: 130px" />
      </a-form-item>
      <a-form-item label="电表2">
        <a-input v-model:value="formData.number_2.current" placeholder="当前记录" style="width: 130px" />
        <span>-</span>
        <a-input v-model:value="formData.number_2.last" placeholder="上次记录" style="width: 130px" />
      </a-form-item>
      <a-form-item label="电表6" >  
        <a-input v-model:value="formData.number_6.current" placeholder="当前记录" style="width: 130px" />
        <span>-</span>
        <a-input v-model:value="formData.number_6.last" placeholder="上次记录" style="width: 130px" />
      </a-form-item>
      <a-form-item :wrapper-col="{ offset: 9, span: 16 }">
        <a-button type="primary"  @click="onSubmit">生成表格</a-button>
      </a-form-item>
    </a-form>
  </div>
  <div class="table" v-else>
    <a-table :dataSource="dataSource" :columns="columns" :pagination="false">
      <template #bodyCell="{ column, record }">
        <template v-if="column.key === 'deg'">
          {{ record.deg }} 度
        </template>
      </template>
    </a-table>
    <div class="btns">
      <a-button type="primary"  @click="dataSource = []">返回</a-button>
    </div>
  </div>
</template>

<script setup>
  import { computed, onMounted, reactive,ref } from 'vue';
  import { localStorage } from "@/utils/localStorage"
  import debounce from "@/utils/debounce"
  const dataSource = ref([])
  const columns = ref([
    {
      title: '房号',
      dataIndex: 'name',
      key: 'name',
    },
    {
      title: '电表当前记录',
      dataIndex: 'current',
      key: 'current',
    },
    {
      title: '电表上次记录',
      dataIndex: 'last',
      key: 'last',
    },
    {
      title: '用电量',
      dataIndex: 'deg',
      key: 'deg',
    },
    {
      title: '电费',
      dataIndex: 'sum',
      key: 'sum',
    },
    {
      title: '公摊电费',
      dataIndex: 'public',
      key: 'public',
    },
    {
      title: '总计',
      dataIndex: 'total',
      key: 'total',
    }
  ])
  const cache_formData = localStorage.get('formData')
  const formData = reactive({
    number:4,
    money: 100,
    number_3:{
      current:0,
      last:0,
    },
    number_1:{
      current:0,
      last:0,
    },
    number_2:{
      current:0,
      last:0,
    },
    number_6:{
      current:0,
      last:0,
    },
  });
  onMounted(() => {
    if(!!cache_formData){
      formData.number_3.last = cache_formData.number_3.current
      formData.number_1.last = cache_formData.number_1.current
      formData.number_2.last = cache_formData.number_2.current
      formData.number_6.last = cache_formData.number_6.current
    }
  })
  const result = computed(() => {
    let money = Number(formData.money);
    let number = Number(formData.number);
    let number_3 = Number(formData.number_3.current) - Number(formData.number_3.last);
    let number_3_price = number_3 * 0.6;
    let number_1 = Number(formData.number_1.current) - Number(formData.number_1.last);
    let number_1_price = number_1 * 0.6;
    let number_2 = Number(formData.number_2.current) - Number(formData.number_2.last);
    let number_2_price = number_2 * 0.6;
    let number_6 = Number(formData.number_6.current) - Number(formData.number_6.last);
    let number_6_price = number_6 * 0.6;
    let public_price = (money - number_3_price - number_1_price - number_2_price - number_6_price) / number;
    let number_3_sum = number_3_price + public_price;
    let number_1_sum = number_1_price + public_price;
    let number_2_sum = number_2_price + public_price;
    let number_6_sum = number_6_price + public_price;
    return {
      number_3,
      number_3_price,
      number_3_sum,
      number_1,
      number_1_price,
      number_1_sum,
      number_2,
      number_2_price,
      number_2_sum,
      number_6,
      number_6_price,
      number_6_sum,
      public_price,
    }
  })
 
  const onSubmit = () => {
    console.log('Success:', formData);
    localStorage.set('formData', formData);
    dataSource.value = [
      {
        name: '03',
        last: formData.number_3.last,
        current: formData.number_3.current,
        deg: Math.round(result.value.number_3),
        sum:result.value.number_3_price.toFixed(2),
        total:result.value.number_3_sum.toFixed(2),
        public:result.value.public_price.toFixed(2),
      },
      {
        name: '01',
        last: formData.number_1.last,
        current: formData.number_1.current,
        deg:Math.round(result.value.number_1),
        sum:result.value.number_1_price.toFixed(2),
        total:result.value.number_1_sum.toFixed(2),
        public:result.value.public_price.toFixed(2),
      },
      {
        name: '02',
        last: formData.number_2.last,
        current: formData.number_2.current,
        deg: Math.round(result.value.number_2),
        sum:result.value.number_2_price.toFixed(2),
        total:result.value.number_2_sum.toFixed(2),
        public:result.value.public_price.toFixed(2),
      },
      {
        name: '06',
        last: formData.number_6.last,
        current: formData.number_6.current,
        deg: Math.round(result.value.number_6),
        sum:result.value.number_6_price.toFixed(2),
        total:result.value.number_6_sum.toFixed(2),
        public:result.value.public_price.toFixed(2),
      },
    ]
  }
</script>

<style scoped lang="scss">
  .box{
    display: flex;
    align-items: center;
    span{
      margin: 0 15px;
    }
    .result{
      display: flex;
      align-items: center;
      margin-left: 20px;
      flex: 1;
      min-width: 0;
      white-space: nowrap;
    }
  }
  .form{
    margin: 0 auto;
    width: 500px;
    h3{
      margin-bottom: 30px;
      text-align: center;
    }
    span{
      margin: 0 10px;
    }
  }
  .table{
    margin: 0 auto;
    width: 980px;
    .btns{
      margin-top: 30px;
      display: flex;
      justify-content: center;
    }
  }
</style>
