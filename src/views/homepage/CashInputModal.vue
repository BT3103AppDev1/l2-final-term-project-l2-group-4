<template>
  <div
    class="modal-overlay"
    :class="{ active: isOpened }"
    @click="closeModal"
  ></div>
  <div class="modal" v-if="isOpened">
    <div class="investment">
      <text class="addInvestment">Cash</text>
      <button @click="close" class="buttonBgFit">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
          width="24"
          height="24"
        >
          <path
            fill="currentColor"
            d="M6.707 6.293c-.39-.39-1.023-.39-1.414 0-.39.39-.39 1.023 0 1.414l4.793 4.793-4.793 4.793c-.39.39-.39 1.023 0 1.414.39.39 1.023.39 1.414 0l4.793-4.793 4.793 4.793c.39.39 1.023.39 1.414 0 .39-.39.39-1.023 0-1.414l-4.793-4.793 4.793-4.793c.39-.39.39-1.023 0-1.414-.39-.39-1.023-.39-1.414 0l-4.793 4.793-4.793-4.793z"
          />
        </svg>
      </button>
    </div>
    <div class="formDiv">
      <div class="investment">
        <div class="forms">
          <label>Bank Name</label>
          <div class="selectButtons">
            <button @click="openDropDown" class="buttonBg">
              {{ selectedInvestment ? selectedInvestment : "Select Bank Name" }}
            </button>
          </div>
          <div v-if="dropDown">
            <div class="investments">
              <button class="buttonBg" @click="selectInvestment('POSB')">
                POSB
              </button>
            </div>
            <div class="investments">
              <button class="buttonBg" @click="selectInvestment('OCBC')">
                OCBC
              </button>
            </div>
          </div>
        </div>
        <div class="forms">
          <div class="forms">
            <label>Cash Holdings (S$)</label>
            <input
              v-model="amount"
              type="number"
              placeholder="Enter total holdings"
              class="selectInputs"
            />
          </div>
          <div class="forms">
            <label class="inputDiv">Purchase date</label>
            <input
              v-model="purchaseDate"
              type="date"
              placeholder="Enter purchase date"
              class="selectInputs"
              @input="validateDate(purchaseDate)"
            />
            <div v-if="dateError" class="error">{{ dateError }}</div>
          </div>
        </div>
      </div>
      <div class="nextButton">
        <button class="whiteButton" @click="submit">Submit</button>
      </div>
      <div v-if="generalError" class="error">
        {{ generalError }}
      </div>
    </div>
  </div>
</template>
<script>
import {
  collection,
  addDoc,
  getFirestore,
  setDoc,
  getDoc,
  doc,
  updateDoc,
} from "firebase/firestore";
import app from "../../firebase";
import { X } from "lucide-react";
import { getAuth, onAuthStateChanged } from "firebase/auth";
const auth = getAuth();
export default {
  props: ["isOpened"],

  components: {
    X,
  },

  data() {
    return {
      dropDown: false,
      selectedInvestment: "",
      amount: "",
      purchaseDate: "",
      dateError: "",
      user: null,
      generalError: "",
    };
  },

  created() {
    onAuthStateChanged(auth, (user) => {
      this.user = user;
    });
  },

  methods: {
    close() {
      this.$emit("isEmitCash", false);
      this.$emit("cashSubmitted", true);
      this.selectedInvestment = "";
      this.amount = "";
      this.purchaseDate = "";
      this.dateError = "";
      this.generalError = "";
      this.dropDown = false;
    },

    openDropDown() {
      this.dropDown = !this.dropDown;
    },

    selectInvestment(investmentType) {
      this.selectedInvestment = investmentType;
      this.dropDown = false;
    },

    validateDate(dateString) {


      if (!dateString.trim()) {
        this.dateError = "";
        return true; // Consider empty input as valid for this specific check
      }
      const inputDate = new Date(dateString);
      const today = new Date();
      today.setHours(0, 0, 0, 0); // Reset today's time to ensure accurate comparison

      if (inputDate > today) {
        this.dateError = "Date must not be in the future.";
        return false;
      } else {
        this.dateError = "";
        return true;
      }
    },

    async submit() {
      if (!this.validateDate(this.purchaseDate)) {
        this.dateError = "Format: DD/MM/YYYY.";
        return; // Exit the submit function early
      }

      if (this.amount === "" || this.purchaseDate === "" || this.selectedInvestment == "") {
        this.generalError = "All fields are required"
        return;
      }

      this.dateError = "";

      if (this.user) {
        const userId = this.user.uid;
        var db = getFirestore(app);
        const docRef = doc(db, "users", userId);
        const data = await getDoc(docRef);
        const realDate = this.purchaseDate.split('-')
        const inputDate = realDate[2] + "/" + realDate[1] + "/" + realDate[0]
        console.log(inputDate);

        if (!data.exists()) {
          await setDoc(docRef, {
            id: userId,
            cash: [
              {
                selectedInvestment: this.selectedInvestment,
                amount: this.amount,
                purchaseDate: inputDate,
              },
            ],
            bonds: [],
            cpf: [],
            stocks: [],
            others: [],
          });
        } else {
          const existing = data.data().cash;
          await updateDoc(docRef, {
            cash: [
              ...existing,
              {
                selectedInvestment: this.selectedInvestment,
                amount: this.amount,
                purchaseDate: inputDate,
              },
            ],
          });
        }

        console.log("successful");
        this.close();
      }
    },
  },
};
</script>
