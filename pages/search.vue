<template>
  <div>
    <LicensePlateSearch />
  </div>
</template>

<script>
import LicensePlateSearch from '~/components/LicensePlateSearch.vue'

export default {
  name: 'SearchPage',
  components: {
    LicensePlateSearch
  },
  head () {
    return {
      title: 'Search by License Plate'
    }
  }
}
</script>