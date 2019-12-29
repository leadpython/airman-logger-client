<template>
  <div class="scan-container">
    <q-btn @click="showIPEdit = true" style="position: absolute; top: 5px; left: 0px;" flat color="primary" label="Set Server IP" />
    <div class="scan-container-half">
      <q-icon v-if="isSuccess" name="done_putline" style="font-size: 20px;" color="success" />
      <q-input outlined v-model="cacid" style="width: 400px" @input="scan" dense />
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
        { name: 'lastName', label: 'Phase', align: 'left', field: 'lastName' },
        { name: 'firstName', label: 'Description', align: 'left', field: 'firstName' },
        { name: 'isIn', label: 'IN/OUT', align: 'left', field: 'isIn' },
        { name: 'date', label: 'Timestamp', align: 'left', field: 'date' }
      ]
    }
  },
  created () {
    this.serverIP = window.localStorage.getItem('airman-logger-admin-ip')
  },
  methods: {
    scan () {
      const self = this
      const serverIP = window.localStorage.getItem('airman-logger-admin-ip')
      axios.post(serverIP).then(response => {
        const { data, status } = response.data
        if (status) {
          const { lastName, firstName, isIn, date } = data.data
          self.scans.unshift({
            lastName, firstName, isIn: isIn ? 'IN' : 'OUT', date: date.toLocaleDateString()
          })
          self.isSuccess = true
        } else {
          self.isFail = true
        }
        self.cacid = ''
      }).catch(error => {
        if (error) {
          console.log(error)
        }
        self.isFail = true
      })
    },
    setServerIP () {
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
</style>
