<template>
  <div>
    <vuci-form uci-config="task1" :key="tableRefresh">
      <vuci-typed-section type="interface" :columns="columns">
        <template #name="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="name" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="address" />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="netmask" />
        </template>
        <template #buttons="{ s }">
          <a-button
            type="primary"
            @click="showEdit(s)"
            v-text="'Edit'"
            style="margin-right: 5px"
          ></a-button>
          <a-button
            type="danger"
            @click="showDelete(s)"
            v-text="'Delete'"
            style="margin-right: 5px"
          >
          </a-button>
        </template>
      </vuci-typed-section>
      <template #footer>
        <div>
          <div style="display: flex">
            <label style="margin-right: 5px; margin-left: 25%; padding-top: 5px">Interface name: </label>
            <a-input
              @pressEnter="validation()"
              v-model="interfaceName"
              style="margin-right: 100px; width: 200px"
              required
            ></a-input>
            <a-button @click="validation()" type="primary" v-text="'Create'"> </a-button>
          </div>
        </div>
      </template>
    </vuci-form>
    <taskModal
      :key='modalRefresh'
      :isVisible='isVisible'
      :sname="sname"
      :interfaceName='interfaceName'
      @onSubmit='onSubmit'
    >
    </taskModal>
  </div>
</template>

<script>
import taskModal from './components/taskModal.vue'
export default {
  components: { taskModal },
  data () {
    return {
      isVisible: false,
      interfaceName: '',
      sname: '',
      availability: true,
      tableRefresh: 50,
      modalRefresh: 0,
      columns: [
        { name: 'name', label: 'Interface Name' },
        { name: 'address', label: 'Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'buttons', label: 'Actions' }
      ]
    }
  },
  methods: {
    validation () {
      if (this.interfaceName.length <= 0) {
        this.$message.error('Please enter interface name!')
      } else if (this.interfaceName.length >= 20) {
        this.$message.error('Interface name is too long (Max = 20char)')
      } else {
        this.checkIfExistInConfig(this.interfaceName)
        if (this.availability === true) {
          this.handleAdd()
        } else {
          this.$message.error('Already exists!')
          this.availability = true
        }
      }
    },
    checkIfExistInConfig (name) {
      this.$uci.load('task1')
      const names = this.$uci.sections('task1')
      for (let i = 0; i < names.length; i++) {
        if (name === names[i].name) {
          this.availability = false
        }
      }
    },
    onSubmit (item) {
      this.isVisible = item
      this.refreshModal()
      this.refreshTable()
      this.interfaceName = ''
    },
    handleAdd () {
      this.sname = this.$uci.add('task1', 'interface')
      this.$uci.set('task1', this.sname, 'name', this.interfaceName)
      this.$uci.set('task1', this.sname, 'protocol', 'static')
      this.isVisible = !this.isVisible
      this.refreshModal()
    },
    showEdit (s) {
      this.refreshModal()
      this.sname = s['.name']
      this.interfaceName = s.name
      this.isVisible = !this.isVisible
    },
    showDelete (s) {
      this.$confirm({
        content: 'Are you sure you want to delete ' + s.name + '  interface?',
        onOk: () => {
          this.handleDelete(s)
        }
      })
    },
    handleDelete (s) {
      const key = 'updatable'
      this.$uci.del('task1', s['.name'])
      this.$uci.save()
      this.$uci.apply('task1')
      this.checkIfExistInConfig(s.name)
      if (this.availability === true) {
        this.$message.loading({ content: 'Deleting Interface!', key })
        setTimeout(() => {
          this.$message.success({ content: 'Interface was succesfully deleted!', key, duration: 2 })
          this.refreshTable()
          this.refreshModal()
        }, 1000)
      } else {
        this.$message.error('There was an error!')
      }
    },
    refreshModal () {
      this.modalRefresh += 1
    },
    refreshTable () {
      this.tableRefresh += 1
    }
  }
}
</script>
