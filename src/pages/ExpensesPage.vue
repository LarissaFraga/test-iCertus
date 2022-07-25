<template>
  <div class="cards" v-if="showCards">
    <div v-for="item in expensesArr" :key="item.id">
      <q-card class="card" flat bordered>
        <q-card-section vertical>
          <q-card-section vertical>
            <div class="text-bold" @click="getExpenses()">
              Description: {{ item.description }}
            </div>
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
      <q-form @submit="onSubmit" @reset="showForm" class="q-gutter-md">
        <q-input
          filled
          v-model="name"
          label="Description *"
          hint="Description"
          lazy-rules
          value="this.val"
          :rules="[(val) => (val && val.length > 0) || 'Please type something']"
        />
        <q-input
          filled
          v-model="category"
          label="Category "
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
          <q-btn
            label="Submit"
            type="submit"
            color="primary"
            @click="onSubmit"
          />
          <q-btn
            label="Cancel"
            type="cancel"
            color="primary"
            flat
            class="q-ml-sm"
            @click="showForm"
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
  setup() {
    const $q = useQuasar();

    const description = ref(null);
    const category = ref(null);
    const valueExpense = ref(null);

    return {
      description,
      category,
      valueExpense,

      onSubmit() {
        if (
          description.value === '' ||
          category.value === '' ||
          valueExpense.value === ''
        ) {
          $q.notify({
            color: 'red-5',
            textColor: 'white',
            icon: 'warning',
            message: 'Fill all the inputs',
          });
        } else {
          $q.notify({
            color: 'green-4',
            textColor: 'white',
            icon: 'cloud_done',
            message: 'Submitted',
          });
        }

        onclose();
      },
    };
  },

  data() {
    return {
      expensesArr: [],
      activeForm: false,
      showCards: true,
    };
  },

  methods: {
    async getExpenses() {
      const response = await axios.get('http://localhost:3000/expenses');
      console.log(response.data);
      this.expensesArr = response.data;
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

    showForm() {
      this.activeForm = false;
      this.showCards = true;
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
