<template>
  <v-app>
    <v-app-bar app dark>Welcome to the Prediction System</v-app-bar>
    <v-content class="container">
      <div v-for="form in forms" :key="form" class="question">
        <v-text-field required outlined v-if="form.type === 'field'" v-model="form.value" :label="form.title" :rules="form.rules"></v-text-field>
        <v-radio-group v-if="form.type === 'choice'" v-model="form.value">
          <div>{{form.title}}</div>
          <v-radio
            v-for="choice in form.choice" :key="choice"
            :label="choice.label"
            :value="choice.val"
          ></v-radio>
        </v-radio-group>
      </div>
      <v-btn dark block @click="sendDataJSON()">Predict</v-btn>
    </v-content>
    <div class="notifContainer" v-if="notifShow">
      <div class="notifDiv">
        <div v-if="result == 1">
          Customer ini BERPOTENSI untuk menerima tawaran asuransi kendaraan
        </div>
        <div v-if="result == 0">
          Customer ini TIDAK CUKUP BERPOTENSI untuk menerima tawaran asuransi kendaraan
        </div>
        <div class="buttonDone">
          <v-btn dark @click="doneClicked()">done</v-btn>
        </div>
      </div>
    </div>
  </v-app>
</template>

<script>
export default {
  data: () => ({
    forms: [
      {
        title: 'Gender',
        type: 'choice',
        choice: [{label: 'Male', val: 1}, {label: 'Female', val: 0}],
        rules: [(value) => !!value || 'Required'],
        value: null,
        formLabel: 'gender'
      },
      {
        title: 'Age',
        type: 'field',
        rules: [(value) => (!!value && value > 0 && value < 150) || 'Age is not valid'],
        value: null,
        formLabel: 'age'
      },
      {
        title: 'Driving License',
        type: 'choice',
        choice: [{label: 'Have', val: 1}, {label: "Don't have", val: 0}],
        rules: [(value) => !!value || 'Required'],
        value: null,
        formLabel: 'driving_license'
      },
      {
        title: 'Region Code',
        type: 'field',
        rules: [(value) => (!!value && value > 0 && value < 53) || 'Region code must be between 1 and 52'],
        value: null,
        formLabel: 'region_code'
      },
      {
        title: 'Previously Insured',
        type: 'choice',
        choice: [{label: 'Yes', val:1}, {label: 'No', val: 0}],
        rules: [(value) => !!value || 'Required'],
        value: null,
        formLabel: 'previously_insured'
      },
      {
        title: 'Vehicle Damage',
        type: 'choice',
        choice: [{label: 'Yes', val: 1}, {label: 'No', val: 0}],
        rules: [(value) => !!value || 'Required'],
        value: null,
        formLabel: 'vehicle_damage'
      },
      {
        title: 'Annual Premium',
        type: 'field',
        rules: [(value) => (!!value && value >= 0) || 'Annual Premium is not valid'],
        value: null,
        formLabel: 'annual_premium'
      },
      {
        title: 'Channel Binned',
        type: 'field',
        rules: [(value) => (!!value && value > 0) || 'Channel Binned is not valid'],
        value: null,
        formLabel: 'channel_binned'
      },
      {
        title: 'Vehicle Age',
        type: 'choice',
        choice: [{label: '< 1 year', val: 'new'}, {label: '1-2 year', val: 'mid'}, {label: '> 2 year', val: 'old'}],
        rules: [(value) => !!value || 'Required'],
        value: null
      }
    ],
    notifShow: false,
    result: null
  }),
  methods: {
    sendData() {
      var bodyFormData = new FormData();
      bodyFormData['tes'] = 'haii';
      console.log(bodyFormData['tes'])
      this.forms.forEach(val => {
        if (val.title !== 'Vehicle Age') {
          bodyFormData[val.formLabel] = val.value
        }
        else {
          var yearValue = ['new', 'mid', 'old'];
          yearValue.forEach(val => {
            bodyFormData[val] = 0
          })
          bodyFormData[val.value] = 1;
        }
      })
      this.$axios.post('http://127.0.0.1:5000/predict', bodyFormData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(res => {
        console.log(res)
      }).catch(err => {
        console.log(JSON.stringify(err))
      })
    },
    sendDataJSON() {
      var bodyRequest = {};
      this.forms.forEach(val => {
        if (val.title !== 'Vehicle Age') {
          if (val.title !== 'Region Code' && val.title !== 'Annual Premium') {
            bodyRequest[val.formLabel] = parseInt(val.value)
          }
          else {
            bodyRequest[val.formLabel] = parseFloat(val.value)
          }
        }
        else {
          var yearValue = ['new', 'mid', 'old'];
          yearValue.forEach(value => {
            bodyRequest[value] = 0
          })
          yearValue[val.value] = 1
        }
      })
      console.log(JSON.stringify(bodyRequest));
      this.$axios.post('http://127.0.0.1:5000/json_predict', {
        bodyRequest
      }).then(res => {
        console.log(JSON.stringify(res))
        this.notifShow = true
        this.result = res.data
      }).catch(err => alert(err))
    },
    doneClicked() {
      for (let i=0; i<this.forms.length; i++) {
        this.forms[i].value = null
      }
      this.result = null;
      this.notifShow = false
    }
  }
}
</script>

<style>
.container {
  width: 70vw;
  margin-left: 15vw;
}
.question {
  background-color: lightgoldenrodyellow;
  padding: 20px;
  margin: 20px;
  border-radius: 10px;
  border: 1px solid orange;
}
.notifContainer {
  position: fixed;
  z-index: 10;
  height: 100vh;
  width: 100vw;
}
.notifDiv {
  background-color: orange;
  height: 30vh;
  width: 80vw;
  text-align: center;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding-top: 30px;
  font-weight: bold;
  border-radius: 30px;
  font-size: 20px;
}
.buttonDone {
  margin-top: 50px;
}
</style>