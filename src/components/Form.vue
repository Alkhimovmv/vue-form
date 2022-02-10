<template>
  <div class="form-group">
    <div>
      <label for="fio">ФИО</label>
      <input type="text"
        id="fio"
        class="form-control"
        v-model="fio"
        required>
      <span v-if="errors.fio" class="error">{{errors.fio}}</span>
    </div>
    <div>
      <label for="date">Дата рождения</label>
      <Datepicker v-model="date" format="yyyy-MM-dd"></Datepicker>
      <span v-if="errors.date" class="error">{{errors.date}}</span>
    </div>
    <div class="specialization">
      <label for="specialization">Специализация</label>
      <select id="specialization" v-model="selectedSpecialization" required>
        <option v-for="specialization in specializations" :value="specialization.code" v-bind:key="specialization.code">
          {{ specialization.value }}
        </option>
      </select>
      <span v-if="errors.selectedSpecialization" class="error">{{errors.selectedSpecialization}}</span>
    </div>
    <button type="button" @click="onSubmit" class="btn btn-dark">
      Отправить
    </button>

    <table class="table mt-5">
      <thead>
        <tr>
          <th scope="col">ID</th>
          <th scope="col">ФИО</th>
          <th scope="col">Дата рождения</th>
          <th scope="col">Код специализации</th>
          <th scope="col">Важность</th>
          <th scope="col">Удалить</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(entry, i) in allRows" :key="i">
          <th scope="row">{{ i }}</th>
          <td>{{ entry.fio }}</td>
          <td>{{ entry.date }}</td>
          <td>{{ entry.code }}</td>
          <td>{{ entry.important }}</td>
          <td>
            <button type="button" class="text-center button btn-xs btn-danger" v-on:click="removeRow(i)">-</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios'
import Datepicker from 'vuejs-datepicker'
import dateformat from 'dateformat'
export default {
  components: { Datepicker },
  name: 'Form',
  data () {
    return {
      fio: '',
      date: '',
      specializations: [],
      selectedSpecialization: '',
      important: '',
      allRows: [],
      errors: []
    }
  },
  selected: {
    specialization: []
  },
  mounted () {
    this.getSpecializations()
  },

  methods: {
    getSpecializations () {
      axios.post('https://pp.credit/Services/BrokerService/api/Application/Dictionary', {'dictName': 'ARM_OccupationArea'})
        .then((response) => {
          console.log(response.data)
          this.specializations = response.data.description
        })
        .catch(response => {
          console.log('error')
        })
    },
    onSubmit () {
      if (!this.checkForm()) return
      this.allRows.push({ fio: this.fio, date: dateformat(this.date, 'yyyy-mm-dd'), code: this.selectedSpecialization, important: Math.floor(Math.random() * 3) + 1 })
      console.log(this.allRows)
      this.clearForm()
    },
    clearForm () {
      this.fio = ''
      this.date = ''
      this.selectedSpecialization = ''
      this.important = ''
      this.errors = []
    },
    validateFio (value) {
      if (/^[А-я]+(?:\s[А-я]+)+$/.test(value) || !value) {
        this.errors['fio'] = ''
      } else {
        this.errors['fio'] = 'Неверные данные'
      }
    },
    checkForm: function (e) {
      if (this.selectedSpecialization && this.date) {
        return true
      }

      this.errors = []

      if (!this.selectedSpecialization) {
        this.errors['selectedSpecialization'] = 'Не выбрана специализация'
      }
      if (!this.date) {
        this.errors['date'] = 'Не выбрана дата рождения'
      }

      if (/^[А-я]+(?:\s[А-я]+)+$/.test(this.fio)) {
        this.errors['fio'] = ''
      } else if (!this.fio) {
        this.errors['fio'] = 'Поле не заполнено'
      } else {
        this.errors['fio'] = 'Неверные данные'
      }
    },
    removeRow: function (index) {
      this.allRows.splice(index, 1)
    }
  },
  watch: {
    selectedContinent: function () {
      this.specializations = []
      this.selectedSpecialization = ''
    },
    fio (value) {
      this.fio = value
      this.validateFio(value)
    }
  }
}
</script>

<style scoped>
.center {
  width: 50%;
  margin: 0 auto;
}
.form-group {
  display: flex;
  flex-direction: column;
  align-items: baseline;
}
.specialization {
  display: grid;
  margin-bottom: 12px;
}
.error {
  color: red;
}
</style>
