<template>
  <q-page class="flex flex-center">
    <div class="row justify-center items-center full-width q-py-lg">
      <q-card bordered dense flat class="col-8 q-pa-md">
        <div class="row q-col-gutter-md">
          <div class="col-12 text-center q-pb-md">
            <span class="text-h5 text-bold text-primary">Kreditni hisoblash</span>
          </div>
          <div class="col-4">
            <q-select v-model="bean.types_id" :options="types" outlined dense label="Kredit turi" map-options emit-value
                      @update:model-value="update"
                      option-label="name" option-value="id"/>
          </div>
          <div class="col-4">
            <q-input v-model="bean.amount" label="Kredit miqdori" dense outlined :suffix="currency.name"
                     @change="updateAmountSlider()" :mask="mask" unmasked-value reverse-fill-mask
                     :maxlength="mask.length"
            />
            <q-slider v-model="amount" :min="type.min_amount"
                      :max="type.max_amount"
                      :step="type.amount_step"/>
          </div>
          <div class="col-4">
            <q-input v-model="bean.term" label="Kredit muddati" dense outlined suffix="Oy"
                     @change="updateTermSlider()"/>
            <q-slider v-model="term" :min="type.min_term"
                      :max="type.max_term"
                      :step="1"/>
          </div>
        </div>
        <div class="row q-col-gutter-md q-gutter-y-md">
          <div class="col-4">
            <q-input v-model="bean.annual_interest_rate" label="Yillik foiz stavkasi" dense outlined suffix="%"
                     @change="updateAnnual"
                     type="number"
                     step="any"/>
          </div>
          <div class="col-4">
            <q-select v-model="bean.is_annuity" :options="calc_types" outlined dense label="Kredit turi" map-options
                      emit-value
                      option-label="name" option-value="id"/>
          </div>
          <div v-if="bean.types_id === 7" class="col-4">
            <q-select v-model="bean.currency_id" :options="currencies" map-options emit-value option-label="name"
                      @update:model-value="currencyChange" option-value="id" dense label="Valyuta turi" outlined/>
          </div>
        </div>
        <div class="row q-col-gutter-md q-gutter-y-md">
          <div class="col-12">
            <span class="text-h6 text-uppercase text-primary">Qo'shimcha xarajatlar</span>
          </div>
          <div class="col-4">
            <q-input v-model="bean.insurance" label="Sug'urta xaarajatlari" dense outlined suffix="uzs" :mask="mask"
                     unmasked-value reverse-fill-mask :maxlength="mask.length"/>
          </div>
          <div class="col-4">
            <q-input v-model="bean.notary" label="Notarius xaarajatlari" dense outlined suffix="uzs" :mask="mask"
                     unmasked-value reverse-fill-mask :maxlength="mask.length"/>
          </div>
          <div class="col-4">
            <q-input v-model="bean.valuation_of_collateral" label="Garovni baholash bo'yicha хarajatlar" dense outlined
                     suffix="uzs" :mask="mask" unmasked-value reverse-fill-mask :maxlength="mask.length"/>
          </div>
          <div class="col-4">
            <q-input v-model="bean.other" label="Boshqa хarajatlar" dense outlined suffix="uzs" :mask="mask"
                     unmasked-value reverse-fill-mask :maxlength="mask.length"/>
          </div>
          <div class="col-4">
            <q-input v-if="bean.types_id === 7" v-model="bean.commission" label="Komission to'lov" dense outlined
                     suffix="%" type="number"
                     step="any" @change="updateCommission"/>
          </div>
          <div class="col-4 flex items-center justify-end">
            <q-btn color="primary" @click="onSubmit" :loading="loading">Hisoblash</q-btn>
          </div>
        </div>
        <div v-if="showInfo">
          <!-- start info section -->
          <div class="row items-center justify-center bg-primary header-section">
            <div class="col-12 text-center text-white text-bold q-py-md">Kredit ma'lumotlari</div>
            <div class="col-6"><span>Kredit nomi:</span>{{ cred_info.type_name }}</div>
            <div class="col-2"><span>Muddati:</span>{{ cred_info.term }} oy</div>
            <div class="col-2">
              <span>yillik foiz stavkasi:</span>
              {{ cred_info.annual_interest_rate ? cred_info.annual_interest_rate : 0 }} %
            </div>
<!--            <div class="col-2"><span>Kredit to'liq qiymati:</span>{{ cred_info.full_amount }} %</div>-->
          </div>
          <!-- end info section -->
          <div class="row flex justify-between q-mt-lg q-px-lg">
            <span class="text-bold text-h4 text-primary">Kredit ma'lumotlari</span>
            <q-btn color="primary" @click="clear" :loading="loading">Bekor qilish</q-btn>
          </div>
          <!-- start table section -->
          <table class="table_info">
            <tr class="bg-primary">
              <th>Oy</th>
              <th>Kredit balansi</th>
              <th>Asosiy qarz</th>
              <th>Kommison to'lov</th>
              <th>Uchinchi shaxslar bilan bog'liq xarajatlar</th>
              <th>Boshqa xarajatlar</th>
              <th>Foiz</th>
              <th>Jami oylik to'lov</th>
            </tr>
            <tr v-for="item in data">
              <td>{{ item.month }}</td>
              <td>{{ number_format(item.credit_balance, 2) }}</td>
              <td>{{ number_format(item.principal_debt, 2) }}</td>
              <td>{{ number_format(item.commission, 2) }}</td>
              <td>{{ number_format(item.third_party_costs, 2) }}</td>
              <td>{{ number_format(item.other_costs, 2) }}</td>
              <td>{{ number_format(item.percentage_debt, 2) }}</td>
              <td>{{ number_format(item.total_monthly_payment, 2) }}</td>
            </tr>
            <tr>
              <td><strong>Jami</strong></td>
              <td></td>
              <td>{{ number_format(data.reduce((sum, item) => sum + item.principal_debt, 0), 2) }}</td>
              <td></td>
              <td></td>
              <td></td>
              <td>{{ number_format(data.reduce((sum, item) => sum + item.percentage_debt, 0), 2) }}</td>
              <td>{{ number_format(data.reduce((sum, item) => sum + item.total_monthly_payment, 0), 2) }}</td>
            </tr>
          </table>
          <!-- end table section -->
        </div>
      </q-card>


    </div>
  </q-page>
</template>

<script setup>
import {onMounted, ref, watch} from "vue";
import {$axios} from "boot/axios";

const apiUrl = "credit-calc";

const mask = "### ### ### ### ### ###"

const bean = ref({
  types_id: 1,
  amount: null,
  term: null,
  annual_interest_rate: 1,
  is_annuity: 1,
  currency_id: 1,
  insurance: 0,
  notary: 0,
  valuation_of_collateral: 0,
  other: 0,
  commission: 0
})

const beanDefault = ref({
  types_id: 1,
  amount: null,
  term: null,
  annual_interest_rate: 1,
  is_annuity: 1,
  currency_id: 1,
  insurance: 0,
  notary: 0,
  valuation_of_collateral: 0,
  other: 0,
  commission: 0
})

const showInfo = ref(false)

const cred_info = ref({
  type_name: null,
  term: null,
  annual_interest_rate: null,
  full_amount: null
})

const amount = ref(0);

const term = ref(0);

const data = ref([])

const types = ref([
  {
    id: 1,
    name: 'Onlayn mikroqarz',
    min_amount: 1_000_000,
    max_amount: 50_000_000,
    amount_step: 1_000_000,
    min_term: 3,
    max_term: 36
  },
  {
    id: 2,
    name: 'Iste\'mol kreditlari',
    min_amount: 1_000_000,
    max_amount: 200_000_000,
    amount_step: 1_000_000,
    min_term: 3,
    max_term: 36
  },
  {
    id: 3,
    name: 'Mikroqarz',
    min_amount: 1_000_000,
    max_amount: 100_000_000,
    amount_step: 1_000_000,
    min_term: 3,
    max_term: 36
  },
  {
    id: 4,
    name: 'Mikroqarz "Drive"',
    min_amount: 1_000_000,
    max_amount: 50_000_000,
    amount_step: 1_000_000,
    min_term: 3,
    max_term: 24
  },
  {
    id: 5,
    name: 'Quyosh panellarini sotib olish va o‘rnatish uchun iste\'mol krediti',
    min_amount: 1_000_000,
    max_amount: 60_000_000,
    amount_step: 1_000_000,
    min_term: 3,
    max_term: 84
  },
  {
    id: 6,
    name: 'Ta\'lim kreditlari',
    min_amount: 1_000_000,
    max_amount: 60_000_000,
    amount_step: 1_000_000,
    min_term: 3,
    max_term: 84
  },
  {
    id: 7,
    name: 'Tadbirkorlik sub\'ektlari uchun',
    min_amount: 100,
    max_amount: 1_000_000_000_000,
    amount_step: 100,
    min_term: 3,
    max_term: 60
  },
])

const type = ref({})

const calc_types = ref([
  {
    id: 1,
    name: 'Oddiy'
  },

  {
    id: 2,
    name: 'Annuitet'
  }
])

const currencies = ref([
  {
    id: 1,
    name: 'uzs'
  },
  {
    id: 69,
    name: 'usd'
  },
  {
    id: 21,
    name: 'eur'
  }
])

const currency = ref({})

const loading = ref(false)




function update(opt) {
  type.value = types.value.filter(el => el.id === opt)[0];
  bean.value.amount = type.value.min_amount;
  amount.value = type.value.min_amount;
  term.value = type.value.min_term;
  currency.value = currencies.value[0];
  bean.value.currency_id = currency.value.id;
}

function currencyChange(opt) {
  currency.value = currencies.value.filter(el => el.id === opt)[0];
  bean.value.currency_id = currency.value.id;
}

function updateAmountSlider() {
  if (bean.value.amount < type.value.min_amount) {
    bean.value.amount = type.value.min_amount;
  } else if (bean.value.amount > type.value.max_amount) {
    bean.value.amount = type.value.max_amount;
  } else {
    console.log(Math.round(bean.value.amount / type.value.amount_step) * type.value.amount_step);
    bean.value.amount = Math.round(bean.value.amount / type.value.amount_step) * type.value.amount_step;
  }
  amount.value = bean.value.amount;
}

function updateTermSlider() {
  if (bean.value.term < type.value.min_term) {
    bean.value.term = type.value.min_term;
  } else if (bean.value.term > type.value.max_term) {
    bean.value.term = type.value.max_term;
  }
  term.value = bean.value.term;
}

function updateAnnual() {
  if (bean.value.annual_interest_rate < 1) {
    bean.value.annual_interest_rate = 1;
  } else if (bean.value.annual_interest_rate > 100) {
    bean.value.annual_interest_rate = 100;
  }
}

function updateCommission() {
  if (bean.value.commission < 0) {
    bean.value.commission = 0;
  } else if (bean.value.commission > 100) {
    bean.value.commission = 100;
  }
}

function number_format(number, decimals) {
  return number !== null
    ? number
      .toFixed(decimals >= 0 ? decimals : 2)
      .toString()
      .replace(
        /(\d)(?=(\d{3})+(?!\d))/g,
        "$1" + " "
      )
    : null;
}


function onSubmit() {
  loading.value = true
  data.value = []
  $axios.post(apiUrl, bean.value).then(res => {
    if (res.data.status) {
      cred_info.value.type_name = type.value.name;
      cred_info.value.term = bean.value.term;
      cred_info.value.annual_interest_rate = bean.value.annual_interest_rate;
      data.value.splice(0, data.value.length, ...res.data.data);
      showInfo.value = true;
    }else {
      alert(res.data.message);
    }
  }).catch(err => console.error(err))
    .finally(() => {
      loading.value = false
    });
}

function clear(){
  bean.value = beanDefault.value;
  currency.value = currencies.value[0];
  type.value = types.value[0]
  bean.value.types_id = types.value[0].id;
  bean.value.amount = type.value.min_amount;
  amount.value = type.value.min_amount;
  term.value = type.value.min_term;
  bean.value.is_annuity = calc_types.value[0].id;
  showInfo.value = false;
}

watch(() => amount.value, () => {
  bean.value.amount = amount.value
})

watch(() => term.value, () => {
  bean.value.term = term.value
})

onMounted(() => {
  clear()
})
</script>

<style lang="scss">
//  start info section
.header-section {
  padding: 24px 128px 48px;
  margin-top: 64px;

  div {
    display: flex;
    flex-direction: column;
    color: #fff;
    font-size: 28px;
    letter-spacing: 0.6px;


    &:first-child {
      font-size: 32px;
    }

    span {
      font-size: 18px;
      letter-spacing: 0;
    }
  }
}

//  end info section

//  start table section
.table_info {
  margin-top: 28px;
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

.table_info td, .table_info th {
  padding: 12px;
  text-align: center;
  max-width: 160px;
}

.table_info tr:nth-child(even) {
  background-color: #f2f2f2;
}

.table_info tr:hover {
  background-color: #ddd;
}

.table_info th {
  padding: 4px 12px;
  color: white;
}

//  end table section

</style>
