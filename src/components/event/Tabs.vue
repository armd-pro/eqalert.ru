<template>
  <b-tabs>
    <template slot="tabs">
      <b-nav-item v-for="tab in tabs"
        :key="tab.href"
        :active="isTabActive(tab)"
        :href="tab.href"
        @click="onTabSwitch(tab)"
        v-if="isTabAvailable(tab)">
        {{ tab.text }}
      </b-nav-item>
    </template>
  </b-tabs>
</template>

<script>
  export default {
    props: ['event'],
    data() {
      return {
        activeTab: this.$router.currentRoute.fullPath,
        tabs: {
          generalInformation: {
            available: true,
            href: '#',
            text: 'Общая информация'
          },
          settlements: {
            available: false,
            href: '#',
            text: 'Ближайшие населенные пункты'
          },
          buildings: {
            available: false,
            href: '#',
            text: 'Здания и сооружения'
          },
          momentTensor: {
            available: false,
            href: '#',
            text: 'Тензор момента'
          },
          ldos: {
            available: false,
            href: '#',
            text: 'Магистральные объекты'
          }
        }
      }
    },
    methods: {
      convertTabName: function(tab) {
        if (!tab) return 'generalInformation'
        if (tab === 'moment-tensor') return 'momentTensor'
        return tab
      },
      isTabActive: function(tab) {
        return this.activeTab === tab.href.substr(1)
      },
      isTabAvailable: function(tab) {
        // Add user access rights here in the future.
        return tab.available
      },
      getHref: function(tab) {
        let params = { id: this.event.id }

        if (tab) params.tab = tab

        return this.$router.resolve({ name: 'Event', params: params }).href
      },
      onTabSwitch: function(object) {
        const tab = Object.keys(this.tabs).find(key => this.tabs[key].href === object.href)

        this.setActiveTab(tab)

        // Send event back to the parent component.
        this.$emit('onTabSwitch', tab)
      },
      setActiveTab: function(tab = this.$router.currentRoute.params.tab) {
        this.activeTab = this.tabs[this.convertTabName(tab)].href.substr(1)
      },
      setAvailability: function() {
        this.tabs.buildings.available = this.event.has_buildings_msk64_analysis
        this.tabs.ldos.available = this.event.has_long_distance_objects_analysis
        this.tabs.momentTensor.available = this.event.has_mt
        this.tabs.settlements.available = this.event.has_cities_msk64_analysis
      },
      setData: function() {
        this.setAvailability()
        this.setHrefs()

        this.setActiveTab()
      },
      setHrefs: function() {
        this.tabs.buildings.href = this.getHref('buildings')
        this.tabs.generalInformation.href = this.getHref()
        this.tabs.ldos.href = this.getHref('ldos')
        this.tabs.momentTensor.href = this.getHref('moment-tensor')
        this.tabs.settlements.href = this.getHref('settlements')
      }
    },
    created() {
      this.setData()
    },
    watch: {
      event: function() {
        this.setData()
      }
    }
  }
</script>

<style lang="scss">
  .tabs {
    .nav-tabs {
      justify-content: center;

      .nav-item{
        a {
          font-size: 90%;
        }
      }
    }

    .tab-content {
      padding-top: 2%;
    }
  }
</style>