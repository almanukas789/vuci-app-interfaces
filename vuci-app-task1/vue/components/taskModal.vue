<template>
  <div>
    <a-modal v-model="modalVisability" :title="'Interface: ' + interfaceName" :width="450" @cancel='onSubmit()'>
      <template>
        <vuci-form uci-config="task1" @applied="onSubmit()">
          <vuci-named-section
            v-slot="{ s }"
            :name="sname"
            :card="true"
            :columns="modalColumns"
          >
            <vuci-form-item-select
              @change='handleProtocolChange'
              :uci-section="s"
              :initial="'Static'"
              :label="'Protocol'"
              name="protocol"
              :options="protocols"
              required
            />
            <vuci-form-item-input
              depend="protocol === 'static'"
              :uci-section="s"
              :label="'Address'"
              name="address"
              rules="ip4addr"
              :required="true"
            />
            <vuci-form-item-input
              depend="protocol === 'static'"
              :uci-section="s"
              :label="'Netmask'"
              name="netmask"
              rules="netmask4"
              :required="true"
            />
            <vuci-form-item-input
              depend="protocol === 'static'"
              :uci-section="s"
              :label="'Gateway'"
              name="gateway"
              rules="ip4addr"
              :required="true"
            />
            <vuci-form-item-list
              depend="protocol === 'static'"
              :uci-section="s"
              :label="'DNS'"
              name="dns"
              rules="ip4addr"
              :required="true"
            />
          </vuci-named-section>
        </vuci-form>
      </template>
      <template #footer>
        <div></div>
      </template>
    </a-modal>
  </div>
</template>

<script>
export default {
  props: {
    isVisible: Boolean,
    interfaceName: String,
    sname: String
  },
  data () {
    return {
      modalVisability: this.isVisible,
      protocols: [
        ['static', 'Static'],
        ['dhcp', 'DHCP']
      ],
      modalColumns: [
        { name: 'name', label: 'Interface Name' },
        { name: 'protocol', label: 'Protocol' },
        { name: 'address', label: 'Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'gateway', label: 'Gateway' },
        { name: 'dns', label: 'DNS' }
      ]
    }
  },
  methods: {
    onSubmit () {
      this.modalVisability = false
      this.$emit('onSubmit', this.modalVisability)
    },
    handleProtocolChange (self) {
      if (self.model === 'dhcp') {
        this.$uci.set('task1', this.sname, 'dns', undefined)
        this.$uci.set('task1', this.sname, 'address', undefined)
        this.$uci.set('task1', this.sname, 'gateway', undefined)
        this.$uci.set('task1', this.sname, 'netmask', undefined)
        //  random fix
        this.$uci.set('task1', this.sname, 'random', undefined)
      } else {
        this.$uci.reset()
      }
    }
  }
}
</script>
