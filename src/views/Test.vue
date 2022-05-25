<template>
  <div id="app">
    <h2>DataTable loading template issue</h2>
    <p>The #loading template gets rendered the first time with the DataTable
      but not afterwards (eg loading during pagination)
    </p>
    <Button label="Refresh the page" @click="refreshPage()"/>
    <DataTable
      class="myDataTable"
      :value="rows"
      responsiveLayout="scroll"
      :paginator="true"
      :rows="3"
      :lazy="true"
      @page="onPage"
      :totalRecords="totalRecords"
      v-model:first="first"
      :loading="loading"
    >
      <Column field="id" header="id" />
      <Column field="name" header="name" />
      <template #empty>
        No data to display.
      </template>
      <template #loading>
        <div style="text-align: center; width: 90%">
            <ProgressBar mode="indeterminate" class="mb-4" style="height: .5em" />
            <p>Loading your data. Please wait...</p>
            <p>
              <Button label="Cancel" @click="cancelLoading" />
            </p>
          </div>
      </template>
    </DataTable>
  </div>
</template>

<script>
export default {
  data() {
    return {
      abortController: null,
      loading: false,
      totalRecords: 0,
      rows: [],
      first: 0,
      lastFirst: 0
    }
  },
  mounted() {
    this.loadDataAsync()
  },
  methods: {
    async loadDataAsync(first) {
      this.loading = true
      this.abortController = new AbortController()
      try {
        this.rows = await this.fetchRowsWithFakeNetworkAsync(first)
        this.totalRecords = 40
        this.lastFirst = first
      }
      catch (err) {
        this.$toast.add({severity: 'warn', summary: 'Cancelled', detail: err.reason, life: 3000})
        this.first = this.lastFirst
      }
      finally {
        this.abortController = null
        this.loading = false
      }
    },
    async onPage(event) {
      await this.loadDataAsync(event.first)
    },
    fetchRowsWithFakeNetworkAsync(first = 0) {
      return new Promise((resolve, reject) => {
          const timeoutId = setTimeout(() => {
            resolve([
              { id: first, name: "John Doe" },
              { id: first + 1, name: "Jane Doe" },
              { id: first + 2, name: "Pete Doe" }
            ])
          }, 2000)

          this.abortController.signal.addEventListener('abort', event => {
            clearTimeout(timeoutId)
            reject(event.target)
          })
        })
    },
    cancelLoading() {
      this.abortController.abort('Loading has been cancelled.')
    },
    refreshPage() {
      window.location.reload()
    }
  }
}
</script>

<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding: 4rem;
}

.myDataTable :deep(.p-component-overlay) {
  background-color: rgba(255, 255, 255, .75);
}
</style>