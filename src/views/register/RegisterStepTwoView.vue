<template>
  <div class="register-step-two-view">
    <DefaultBox>
      <h2>Complete Your Profile</h2>
      <form>
        <InputContainer :desc="'Name:'">
          <b-form-input
            required
            type="text"
            placeholder="type your name"
            maxlength="30"
            size="30"
            v-model="inputUser.name"
          />
        </InputContainer>
        <ColumnContainer>
          <InputContainer :desc="'Image:'">
            <InputImageBtn :hasArchive="hasArchive" @newFile="processFile()" />
          </InputContainer>
          <InputContainer :desc="'Task:'">
            <b-form-input
              required
              type="text"
              placeholder="type your ocupation"
              maxlength="20"
              size="20"
              v-model="inputUser.task"
            />
          </InputContainer>
          <InputContainer :desc="'Age:'">
            <b-form-input
              style="max-width: 70px"
              required
              type="number"
              placeholder="type your age"
              min="18"
              max="110"
              @blur="fixAge()"
              v-model.number="inputUser.age"
            />
          </InputContainer>
        </ColumnContainer>
        <InputContainer :desc="'Street:'">
          <b-form-input
            required
            type="text"
            placeholder="Type yout street"
            maxlength="37"
            size="37"
            v-model="inputUser.adress.street"
          />
        </InputContainer>
        <div class="cep-city">
          <InputContainer :desc="'CEP:'">
            <b-form-input
              required
              type="text"
              placeholder="00000-000"
              v-mask="'#####-###'"
              v-model="inputUser.adress.cep"
              @blur="processCEP()"
            />
          </InputContainer>
          <InputContainer :desc="'City:'">
            <b-form-input
              required
              type="text"
              placeholder="City - State"
              maxlength="20"
              size="20"
              v-model="inputUser.adress.city"
            />
          </InputContainer>
        </div>
        <div>
          <InputContainer :desc="'Who You Are:'">
            <b-form-input
              required
              type="text"
              placeholder="Type about you..."
              maxlength="180"
              size="180"
              v-model="inputUser.whoIAm"
            />
          </InputContainer>
        </div>
        <div class="submit-btn">
          <b-button @click.prevent="completeProfile()" variant="primary"
            >Done</b-button
          >
        </div>
      </form>
    </DefaultBox>
  </div>
</template>

<script>
import { consultCEP } from "@/plugins/searchCEP.js";
import { getFirestore, setDoc, doc } from "firebase/firestore";
import DefaultBox from "@/components/DefaultBox.vue";
import InputContainer from "@/components/InputContainer.vue";
import ColumnContainer from "@/components/ColumnContainer.vue";
import InputImageBtn from "./InputImageBtn.vue";
export default {
  name: "Register",
  components: {
    DefaultBox,
    InputContainer,
    ColumnContainer,
    InputImageBtn,
  },
  data() {
    return {
      inputUser: {
        name: "",
        age: 18,
        adress: {
          street: "",
          cep: "",
          city: "",
        },
        task: "",
        whoIAm: "",
      },
      hasArchive: false,
    };
  },
  methods: {
    reset() {
      this.inputUser = {
        name: "",
        age: 0,
        adress: {
          street: "",
          cep: "",
          city: "",
          state: "",
        },
        task: "",
        whoIAm: "",
        uid: "",
      };
    },
    processFile() {
      this.hasArchive = true;
    },
    throwError(error) {
      let msg = "";
      switch (error.code) {
        case "auth/email-already-in-use":
          msg = "This e-mail already in use.";
          this.email = ""; // Limpa o input de E-mail
          break;
        default:
          msg = error.message;
      }
      this.$root.$emit("NewNotification", {
        msg,
        type: "danger",
      });
    },
    async processCEP() {
      const dataCEP = await consultCEP(this.inputUser.adress.cep);
      if (dataCEP.data.uf != undefined) {
        this.inputUser.adress.street =
          dataCEP.data.logradouro + " " + dataCEP.data.complemento;
        this.inputUser.adress.city =
          dataCEP.data.localidade + " - " + dataCEP.data.uf;
      }
    },
    async completeProfile() {
      let userData = this.inputUser;
      userData.uid = window.uid;
      // console.log(window.uid);
      const db = getFirestore();
      try {
        await setDoc(doc(db, "users", userData.uid), userData);
        // console.log("Router Register2 -> Home");
        this.$router.push({ name: "home" });
      } catch (e) {
        console.error("Error adding document: ", e);
      }
    },
    fixAge() {
      if (this.inputUser.age < 18) this.inputUser.age = 18;
      if (this.inputUser.age > 110) this.inputUser.age = 110;
    },
  },
};
</script>

<style>
.register-step-two-view {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.link {
  display: flex;
  justify-content: center;
  font-size: 1rem;
  font-weight: 600;
}

.link-btn {
  text-decoration: none;
}

.submit-btn {
  display: flex;
  justify-content: flex-end;
  padding-top: 20px;
}

.cep-city {
  display: flex;
  gap: 10px;
}
</style>