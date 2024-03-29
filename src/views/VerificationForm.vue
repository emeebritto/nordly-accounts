<template>
  <section id="container" onload="requestAuthorization()">
    <img
      class="sendMail_icon"
      :src="imgUrl('illustrations/undraw_confirmation_re_b6q5.svg')"
    />
    <h1 id="step_status">
      A authorization email was send to {{ formData.mail }}
    </h1>
    <p class="obs_note">expires in 5 minutes</p>

    <section v-show="!events.onRequest">
      <button class="btn-form-submit" @click="backForm()">Cancel</button>
    </section>
    <LoadingForm v-if="events.onRequest"></LoadingForm>
  </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import LoadingForm from "@/components/loadingForm.vue";
import store from "@/store";
import storage from "@/common/storage";
import { istatics, nordlyApi } from "@/services";
import { WSCheckMailError } from "@/common/interface";

export default defineComponent({
  name: "VerificationForm",
  components: {
    LoadingForm,
  },
  computed: {
    formData() {
      return store.getters.formData;
    },
    events() {
      return store.getters.events;
    },
  },
  created() {
    this.requestAuthorization();
  },
  sockets: {
    connect() {
      console.log("nordly socket connected");
    },
    authorized({ accessToken }) {
      storage.setToken(accessToken);
      this.$router.push({ name: "myAccount" });
    },
    unauthorized() {
      alert("access was denied.");
      this.$router.push({ name: "Forms" });
    },
  },
  methods: {
    requestAuthorization() {
      const socketCode = storage.get("__SOCKET_CODE");
      const form = {
        username: this.formData.username,
        mail: this.formData.mail,
        socketCode,
      };
      this.$socket.client.emit("checkMail", form, (res: WSCheckMailError) => {
        console.error(`form verification: ${res.msg}`);
        alert(res.msg);
        if (res.error && res.status === 401) {
          this.$router.push({ name: "Forms" });
        }
      });
    },
    imgUrl(path: string): string {
      return istatics.imgUrl({ path });
    },
    backForm() {
      this.$router.push({ name: "Forms" });
    },
  },
});
</script>
<style scoped lang="scss">
@font-face {
  font-family: Padauk-Regular;
  src: url("../assets/fonts/Padauk-Regular.ttf");
}

#step_status {
  margin: 0 25px;
}

.obs_note {
  font-size: 0.9em;
  margin: 10px 0;
  color: #6b6a71;
}

#container {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  font-family: Padauk-Regular;
  font-size: 1.5em;
  background-color: rgb(0 0 0 /60%);
  border: 2px solid;
  border-image: linear-gradient(135deg, #000, #fff, #000) 1;
  width: 440px;
  height: auto;
  margin: 50px auto;
  backdrop-filter: blur(7px);
  -webkit-backdrop-filter: blur(7px);
}

.sendMail_icon {
  width: 200px;
  margin: 50px 0;
}

.msg-error {
  font-size: 0.9em;
  margin: 0 10px;
  color: #c22526;
}

.input-labels {
  margin: 20px 0;
  font-family: "Gill Sans", "Gill Sans MT", Calibri, "Trebuchet MS", sans-serif;
}

.input-data {
  border: none;
  outline: none;
  text-align: center;
  background-color: transparent;
  border-bottom: 2px solid #404250;
  color: #fff;
  padding: 0 10px;
  margin: 10px 0;
  border-radius: 10px;
  width: 330px;
  height: 25px;
}

.input-data:focus {
  border-bottom: 2px solid #858586;
}

.btn-form-submit {
  margin: 20px 10px;
  border: none;
  cursor: pointer;
  color: #e0dfde;
  font-size: 0.8em;
  background-color: transparent;
  padding: 5px 10px;
  border: 2px solid #e0dfde;
  font-family: "Gill Sans", "Gill Sans MT", Calibri, "Trebuchet MS", sans-serif;
  transition: 500ms;
}

.btn-form-submit:hover {
  background-color: #e0dfde;
  color: #000;
}

a {
  text-decoration: none;
  color: #fff;
}
</style>
