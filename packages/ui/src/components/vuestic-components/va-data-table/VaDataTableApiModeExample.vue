<template>
  <va-data-table
    :fields="fields"
    :data="users"
    :loading="loading"
    :total-pages="totalPages"
    api-mode
    @pageSelected="readPage"
  />
</template>

<script>
import users from '../../../data/users.json'
import VaDataTable from './VaDataTable'

export default {
  components: { VaDataTable },
  data () {
    return {
      loading: false,
      perPage: 4,
      users: [],
      fields: [{
        name: 'firstName',
        label: 'First Name',
        width: '30%',
      }, {
        name: 'lastName',
        label: 'Last Name',
        width: '30%',
      }, {
        name: 'country',
        label: 'Country',
      }],
    }
  },
  computed: {
    totalPages () {
      return Math.floor(users.length / this.perPage)
    },
  },
  created () {
    this.readPage()
  },
  methods: {
    readPage (page = 1) {
      const sliceFrom = (page - 1) * this.perPage

      this.loading = true

      setTimeout(() => {
        this.users = users.slice(sliceFrom, sliceFrom + this.perPage)
        this.loading = false
      }, 450)
    },
  },
}
</script>

<style lang="scss">
</style>
