<template>
  <div class="q-pa-md q-gutter-sm">
    <q-btn
      color="primary"
      icon="add"
      label="New"
      @click="
        () => {
          this.activeForm = true;
          this.showCards = false;
        }
      "
    />
  </div>
  <div class="cards" v-if="showCards">
    <div v-for="item in expensesArr" :key="item.id">
      <q-card class="card" flat bordered>
        <q-card-section vertical>
          <q-card-section vertical>
            <div class="text-bold">Description: {{ item.description }}</div>
            <div class="text-bold">Category: {{ item.category }}</div>
            <div class="text-bold">Value: {{ item.value }}</div>
          </q-card-section>

          <q-card-actions horizontal class="justify-around q-px-md">
            <q-btn
              flat
              round
              color="black"
              icon="edit"
              @click="
                () => {
                  showCards = false;
                  activeForm = true;
                  getExpenseById(item.id);
                }
              "
            />
            <q-btn
              flat
              round
              color="black"
              icon="delete"
              @click="deleteExpense(item.id)"
            />
          </q-card-actions>
        </q-card-section>
      </q-card>
    </div>
  </div>
  <div class="positionForm" v-if="activeForm">
    <div class="q-pa-md confirmationForm" style="min-width: 400px">
      <q-form @submit="onSubmit" @reset="hideForm" class="q-gutter-md">
        <q-input
          filled
          v-model="description"
          label="Description *"
          hint="Description"
          lazy-rules
          value="this.val"
          :rules="[(val) => (val && val.length > 0) || 'Please type something']"
        />
        <q-input
          filled
          v-model="category"
          label="Category *"
          hint="Category"
          lazy-rules
          value="this.val"
          :rules="[(val) => (val && val.length > 0) || 'Please type something']"
        />

        <q-input
          filled
          v-model="valueExpense"
          label="Value *"
          hint="Value"
          lazy-rules
          value="this.val"
          :rules="[(val) => (val && val.length > 0) || 'Please type something']"
        />

        <div>
          <q-btn label="Submit" type="submit" color="primary" />
          <q-btn
            label="Cancel"
            type="cancel"
            color="primary"
            flat
            class="q-ml-sm"
            @click="hideForm"
          />
        </div>
      </q-form>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { useQuasar } from 'quasar';
import { ref } from 'vue';

export default {
  name: 'ExpensesPage',

  data() {
    return {
      expensesArr: [],
      activeForm: false,
      showCards: true,
      expenseId: null,
      $q: useQuasar(),

      description: ref(null),
      category: ref(null),
      valueExpense: ref(null),
    };
  },

  methods: {
    async getExpenses() {
      const response = await axios.get('http://localhost:3000/expenses');
      this.expensesArr = response.data;
    },
    async getExpenseById(id) {
      const response = await axios.get(`http://localhost:3000/expenses/${id}`);
      this.description = response.data.description;
      this.category = response.data.category;
      this.valueExpense = response.data.value;
      this.expenseId = response.data.id;
      return response.data;
    },
    async deleteExpense(id) {
      await axios.delete(`http://localhost:3000/expenses/${id}`);
      this.getExpenses();
    },
    async addExpense(expense) {
      await axios.post('http://localhost:3000/expenses', expense);
      this.getExpenses();
    },
    async editExpense(expense) {
      await axios.put(`http://localhost:3000/expenses/${expense.id}`, expense);
      this.getExpenses();
    },

    hideForm() {
      this.activeForm = false;
      this.showCards = true;
      this.description = null;
      this.category = null;
      this.valueExpense = null;
    },

    onSubmit() {
      if (this.expenseId) {
        this.editExpense({
          id: this.expenseId,
          description: this.description,
          category: this.category,
          value: this.valueExpense,
        });
      } else {
        this.addExpense({
          description: this.description,
          category: this.category,
          value: this.valueExpense,
        });
      }

      this.$q.notify({
        color: 'green-4',
        textColor: 'white',
        icon: 'cloud_done',
        message: 'Submitted',
      });

      this.expenseId = null;
      this.hideForm();
    },
  },

  mounted() {
    this.getExpenses();
  },
};
</script>

<style lang="scss" scoped>
.card {
  width: 250px;
  height: 200px;
}

.cards {
  display: flex;
  flex-wrap: wrap;
  justify-content: left;
  margin: 16px;
}

.confirmationForm {
  border: 2px solid gray;
}

.positionForm {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 400px;
  margin: 16px;
}
</style>
