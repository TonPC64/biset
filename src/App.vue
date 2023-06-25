<script setup lang="ts">
import { TrashIcon, PlusCircleIcon } from '@heroicons/vue/24/solid';
import { useClipboard } from '@vueuse/core';

</script>

<template>
  <div class="container max-w-2xl mx-auto py-4">
    <!-- Header -->
    <div class="flex justify-between mb-4 mx-4">
      <div class="flex flex-col items-center justify-center">
        <span class="font-bold text-xl">
          <select id="store" v-model="store" ref="store" class="select w-full max-w-xs mr-4" :class="{'select-error': storeValidate}">
            <option disabled selected value="null">Pick Store</option>
            <option v-for="(t, si) in stores" :key="si" :value="t.value">{{ t.text }}</option>
          </select>
        </span>
      </div>
      <button class="btn btn-warning" @click="reset">Reset</button>
    </div>

    <div class="overflow-x-auto mb-4">
      <!-- List Table -->
      <table class="table table-auto">
        <thead>
          <tr>
            <th>Name</th>
            <th>Type</th>
            <th class="text-right">Price</th>
            <th class="text-center w-12">Action</th>
          </tr>
        </thead>
        <tbody>
          <tr :key="i" v-for="(item, i) in lists" class="hover">
            <td>{{ item.name }}</td>
            <td>
              <select :id="`item_type_${i}`" v-model="item.type" class="select select-xs w-full max-w-xs mr-4">
                <option v-for="(t, si) in types" :key="si" :value="t.value">{{ t.text }}</option>
              </select>
            </td>
            <td class="text-right">{{ formatPrice(item.price) }}</td>
            <td>
              <button class="btn btn-error h-12 w-12" @click="removeItem(i)">
                <TrashIcon class="w-4" />
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Input -->
    <form id="input" ref="input" @submit="addItem">
      <div class="flex gap-4 w-full px-4">
        <input id="input_name" ref="input_name" required v-model="input.name" type="text" placeholder="Name"
          class="input input-bordered w-6/12 max-w-xs" />
        <select id="input_type" required v-model="input.type" class="select select-bordered max-w-xs text-xs w-3/12">
          <option disabled :value="null">Select Type</option>
          <option v-for="(t, i) in types" :key="i" :value="t.value">{{ t.text }}</option>
        </select>
        <input id="input_price" required v-model.number="input.price" step="0.01" type="number" placeholder="Price"
          class="input input-bordered w-3/12 max-w-xs" />
        <button class="btn btn-info h-12 w-12" type="submit">
          <PlusCircleIcon class="w-4" />
        </button>
      </div>
    </form>

    <!-- Report -->
    <div class="my-4 flex justify-center" v-if="lists.length > 0">
      <button class="btn btn-xl btn-accent btn-outline" @click="report">Report & Copy</button>
    </div>

    <table class="table table-auto" v-if="showReport">
      <thead>
        <tr>
          <th>Name</th>
          <th class="text-right">Price</th>
          <th>Type</th>
          <th>Store</th>
        </tr>
      </thead>
      <tbody>
        <tr :key="key" v-for="(item, key) in reportData">
          <td>{{ item.value }}</td>
          <td class="text-right">{{ item.price }}</td>
          <td>{{ optionValue(key as string, types) }}</td>
          <td>{{ optionValue(store || '', stores) }}</td>
        </tr>
      </tbody>
    </table>

    <div v-show="showToast" class="toast toast-center">
      <div class="alert alert-success">
        <span>Report data is copied</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

interface Input {
  name: string
  type: string | null
  price: number | null
}

interface Report {
  value: string
  price: string
  priceSum: number
}

interface ReportGroup {
  [key: string]: Report
}

const defaultInput = {
  type: null,
  price: null,
} as Input;

interface Option {
  text: string
  value: string
}

export default defineComponent({
  data() {
    return {
      store: null,
      input: Object.assign({}, defaultInput),
      lists: [
        // mock data
        // { name: 'foo', type: 'week1', price: 123 },
        // { name: 'foo', type: 'week2', price: 123 },
      ] as Input[],
      types: [
        { text: 'Weekly Meal 1st', value: 'week1' },
        { text: 'Weekly Meal 2nd', value: 'week2' },
        { text: 'Weekly Meal 3rd', value: 'week3' },
        { text: 'Weekly Meal 4th', value: 'week4' },
        { text: 'Weekly Meal 5th', value: 'week5' },
        { text: 'Ingredients', value: 'ingredients' },
        { text: 'Household Items', value: 'household' },
        { text: 'Snack&Drink', value: 'snack_drink' },
        { text: 'Personal items', value: 'personal' },
        { text: 'Frozen food', value: 'frozen' },
        { text: 'Fruits', value: 'fruits' },
      ] as Option[],
      stores: [
        { text: 'Tops', value: 'tops' },
        { text: 'BigC', value: 'bigc' },
        { text: 'Market', value: 'market' },
        { text: '7-11', value: 'seven' },
        { text: 'Watson', value: 'watson' },
        { text: 'Lotus', value: 'lotus' },
        { text: 'Homepro', value: 'homepro' },
        { text: 'Makro', value: 'makro' },
        { text: 'Maxvalue', value: 'maxvalue' },
        { text: 'FoodPanda', value: 'foodpanda' },
        { text: 'Grab', value: 'grab' },
      ] as Option[],
      reportData: {} as ReportGroup,
      showToast: false,
    };
  },
  computed: {
    showReport(): Boolean {
      return Object.keys(this.reportData).length !== 0;
    },
    storeValidate(): Boolean {
      return this.store === null;
    },
  },
  methods: {
    reset() {
      this.lists = [];
      this.input = Object.assign({}, defaultInput);
      this.reportData = Object.assign({}, {} as ReportGroup);
    },
    addItem(e: Event) {
      if (!this.input.name || !this.input.type || !this.input.price) {
        return;
      }
      this.lists.push(Object.assign({}, this.input));
      this.input = Object.assign({}, defaultInput);
      e.preventDefault();

      (this.$refs.input_name as HTMLInputElement).focus();
    },
    removeItem(index: number) {
      this.lists.splice(index, 1);
    },
    formatPrice(p: number | null): String {
      return (p || 0).toLocaleString('th-TH', { minimumFractionDigits: 2 });
    },
    report() {
      // validate store
      const storeSelect = this.$refs.store as HTMLSelectElement;
      if (this.store === null) {
        storeSelect.focus();
        return;
      }

      this.reportData = this.lists.reduce((agg: ReportGroup, item: Input) => {
        if (!item.type) return agg;
        if (!item.price) return agg;

        const key = item.type;

        if (!agg[key]) {
          agg[key] = {
            value: '',
            price: '=',
            priceSum: 0,
          };
        }

        if (agg[key].value != '') agg[key].value += ' ';
        if (agg[key].price != '=') {
          if (item.price >= 0) agg[key].price += '+';
        }

        agg[key].value += item.name;
        agg[key].price += item.price;
        agg[key].priceSum += item.price;

        return agg;
      }, {} as ReportGroup);

      // copy data
      const store = this.optionValue(this.store as string, this.stores);
      const arr = Object.keys(this.reportData).map((key: string): String[] => {
        const value = this.reportData[key];

        return [value.value, value.price, this.optionValue(key, this.types), store];
      });
      const excelData = arr.map(lines => lines.join('\t')).join('\n');
      const { copy } = useClipboard();

      copy(excelData).then(() => {
        this.showToast = true;
        setTimeout(() => {
          this.showToast = false;
        }, 2000);
      });
    },
    optionValue(k: string, o: Option[]): String {
      const t = o.find((i: Option) => {
        return i.value == k;
      });

      return t?.text || '';
    },
  },
});
</script>
