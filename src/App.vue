<script setup>
import { computed, onMounted, ref } from "vue";
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpenses from "./components/IncomeExpenses.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";
import { useToast } from "vue-toastification";

const toast = useToast();

const transactions = ref([
  // { id: number , text: string , amount: + or - number },
]);

onMounted(() => {
  // load transactions from localStorage
  const savedTransactions = JSON.parse(localStorage.getItem("transactions"));
  // if savedTransactions is not undefined
  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

// calculate income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// calculate expense
const expense = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// calculate total
const total = computed(() => {
  return Number(income.value) + Number(expense.value);
});

// add transaction
const addTransaction = (transactionData) => {
  transactions.value.push({
    id: Math.floor(Math.random() * 1000000), // generate unique id for transaction object
    text: transactionData.text,
    amount: transactionData.amount,
  });

  //update transactions
  saveTransactionsToLocalStorage();

  toast.success("Transaction added");
};

// delete transaction
const deleteTransaction = (transactionID) => {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== transactionID
  );

  //update transactions
  saveTransactionsToLocalStorage();

  toast.success("Transaction deleted");
};

// save/update transactions to localStorage
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem("transactions", JSON.stringify(transactions.value));
};
</script>

<template>
  <Header />
  <div class="container">
    <Balance :total="+total" />
    <IncomeExpenses :income="+income" :expense="+expense" />
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="deleteTransaction"
    />
    <AddTransaction @transactionSubmitted="addTransaction" />
  </div>
</template>
