<template>
  <div class="scan-container">
    <q-btn @click="showIPEdit = true" style="position: absolute; top: 5px; left: 0px;" flat color="primary" label="Set Server IP" />
    <div class="scan-container-half">
      <div style="display: flex; justify-content: center; align-items: center; width: 300px; height: 300px;">
        <q-icon v-show="isSuccess" name="done_outline" style="font-size: 200px;" color="green"/>
        <q-icon v-show="isFail" name="block" style="font-size: 200px;" color="red"/>
      </div>
      <q-input outlined v-model="cacid" debounce="200" style="width: 400px" @input="scan" dense type="password" />
    </div>
    <div class="scan-container-half">
      <q-table
        virtual-scroll
        style="width: 100%; height: 100%;"
        :pagination.sync="pagination"
        :rows-per-page-options="[0]"
        row-key="name"
        title="Activity"
        :data="scans"
        :columns="columns"
      />
    </div>

    <q-dialog v-model="showIPEdit" persistent square transition-show="scale" transition-hide="scale">
      <q-card class="bg-white text-black" style="width: 400px">
        <q-card-section>
          <div class="text-h6">Server IP Address</div>
        </q-card-section>
        <q-card-section>
          <div class="add-admin-row">
            <label>IP Address</label>
            <q-input v-model="serverIP" square dense outlined :disable="!isEditMode" />
          </div>
        </q-card-section>

        <q-card-actions align="right" class="bg-white text-primary">
          <q-btn flat label="Close" v-close-popup v-if="!isEditMode" />
          <q-btn flat label="Edit" @click="isEditMode = true" v-if="!isEditMode" />
          <q-btn flat label="Cancel" @click="isEditMode = false" v-if="isEditMode" />
          <q-btn flat label="Finalize" v-close-popup @click="setServerIP" v-if="isEditMode" />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'PagePhase',
  data () {
    return {
      cacid: '',
      scans: [],
      serverIP: '',
      showIPEdit: false,
      isEditMode: false,
      isSuccess: false,
      isFail: false,
      pagination: {
        rowsPerPage: 0
      },
      columns: [
        { name: 'last_name', label: 'Last Name', align: 'left', field: 'last_name' },
        { name: 'first_name', label: 'First Name', align: 'left', field: 'first_name' },
        { name: 'phase', label: 'Phase', align: 'left', field: 'phase' },
        { name: 'isIn', label: 'IN/OUT', align: 'left', field: 'isIn' },
        { name: 'date', label: 'Timestamp', align: 'left', field: 'date', format: val => `${this.dateFormat(new Date(val))}` }
      ]
    }
  },
  created () {
    this.serverIP = window.localStorage.getItem('airman-logger-admin-ip')
  },
  methods: {
    dateFormat (date) {
      if (date === '-----') {
        return date
      }
      var hours = date.getHours()
      var minutes = date.getMinutes()
      var ampm = hours >= 12 ? 'pm' : 'am'
      hours = hours > 12 ? hours - 12 : hours
      minutes = minutes < 10 ? '0' + minutes : minutes
      var strTime = hours + ':' + minutes + ' ' + ampm
      return date.getMonth() + 1 + '/' + date.getDate() + '/' + date.getFullYear() + '  ' + strTime
    },
    scan () {
      const self = this
      const serverIP = window.localStorage.getItem('airman-logger-admin-ip')
      self.isSuccess = false
      self.isFail = false
      clearTimeout(window.window.scanTimeout)
      window.scanTimeout = setTimeout(() => {
        axios.post(`http://${serverIP}/api/scan`, { cacid: self.cacid }).then(response => {
          const { data, status } = response.data
          if (status) {
            const { isIn, phase } = data
            self.scans.unshift({
              last_name: data.last_name, first_name: data.first_name, isIn: isIn ? 'IN' : 'OUT', phase, date: (new Date())
            })
            self.isSuccess = true
          } else {
            self.isFail = true
            self.scans.unshift({
              last_name: 'AIRMAN NOT FOUND!', first_name: '-----', isIn: '------', date: '-----'
            })
          }
          self.cacid = ''
        }).catch(error => {
          if (error) {
            console.log(error)
          }
          self.isFail = true
          self.cacid = ''
        })
      }, 100)
    },
    setServerIP () {
      this.isEditMode = false
      window.localStorage.setItem('airman-logger-admin-ip', this.serverIP)
    }
  }
}
</script>

<style lang="scss">
.scan-container {
  display: flex;
  flex-direction: row;
  .scan-container-half {
    width: 50vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
}
.add-admin-row {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
  label {
    font-size: 12px;
    color: rgb(100,100,100);

  }
}

tr:nth-child(even) {
  background-color: #EBEBEB;
}
</style>
