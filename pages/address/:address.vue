<template>
  <div>
    <v-snackbar v-model="copied">
      {{ $t('address.copied') }}
      <v-btn
        color="red"
        text
        @click="copied = false"
      >
        Close
      </v-btn>
    </v-snackbar>

    <v-row>
      <v-col
        :cols="12"
        :sm="12"
        :md="12"
        :lg="12"
      >
        <v-card>
          <v-card-title class="secondary--text mt-n12">
            {{ $t('address.wallet') }}
          </v-card-title>
          <v-card-text class="pl-0 pr-0">
            <v-divider class="mb-3" />

            <span class="overline pl-6">{{ $t('address.title') }}</span>
            <br>
            <span class="subtitle-1 pl-6 secondary--text">{{ address }}</span>

            <v-btn
              v-clipboard:copy="address"
              v-clipboard:success="copy"
              text
              icon
              style="font-size:22px;"
            >
              <span class="lto-copy" />
            </v-btn>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-row>
      <v-col
        :cols="12"
        :sm="6"
        :md="3"
        :lg="3"
      >
        <v-card>
          <v-card-text>
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <span class="pa-0 overline" v-text="$t('address.balance.regular')" />
                <v-icon small class="ml-1 grey--text" v-on="on">
                  mdi-help-circle
                </v-icon>
              </template>
              {{ $t('address.tooltips.regular') }}
            </v-tooltip>

            <p class="title font-weight-bold secondary--text mt-2 mb-0">
              {{ balance.regular | parseAtomic | parseNumber }}
            </p>
          </v-card-text>
        </v-card>
      </v-col>

      <v-col
        :cols="12"
        :sm="6"
        :md="3"
        :lg="3"
      >
        <v-card>
          <v-card-text>
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <span class="pa-0 overline" v-text="$t('address.balance.generating')" />
                <v-icon small class="ml-1 grey--text" v-on="on">
                  mdi-help-circle
                </v-icon>
              </template>
              {{ $t('address.tooltips.generating') }}
            </v-tooltip>

            <p class="title font-weight-bold secondary--text mt-2 mb-0">
              {{ balance.generating | parseAtomic | parseNumber }}
            </p>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col
        :cols="12"
        :sm="6"
        :md="3"
        :lg="3"
      >
        <v-card>
          <v-card-text>
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <span class="pa-0 overline" v-text="$t('address.balance.available')" />
                <v-icon small class="ml-1 grey--text" v-on="on">
                  mdi-help-circle
                </v-icon>
              </template>
              {{ $t('address.tooltips.available') }}
            </v-tooltip>

            <p class="title font-weight-bold secondary--text mt-2 mb-0">
              {{ balance.available | parseAtomic | parseNumber }}
            </p>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col
        :cols="12"
        :sm="6"
        :md="3"
        :lg="3"
      >
        <v-card>
          <v-card-text>
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <span class="pa-0 overline" v-text="$t('address.balance.effective')" />
                <v-icon small class="ml-1 grey--text" v-on="on">
                  mdi-help-circle
                </v-icon>
              </template>
              {{ $t('address.tooltips.effective') }}
            </v-tooltip>

            <p class="title font-weight-bold secondary--text mt-2 mb-0">
              {{ balance.effective | parseAtomic | parseNumber }}
            </p>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-row>
      <v-col
        :cols="12"
        :sm="12"
        :md="12"
        :lg="12"
      >
        <v-card>
          <v-card-title class="secondary--text">
            <span class="mr-2 lto-transactions" />
            {{ $t('address.transactions') }}

            <v-spacer />

            <div style="width:120px;">
              <span class="body-2 grey--text mb-3 mr-2">{{ $t('explorer.type') }}</span>

              <v-select
                v-model="txType"
                label="All"
                :items="txFilter"
                outlined
                single-line
                cache-items
                color="secondary"
                item-color="secondary"
                dense
                class="body-3 pa-0 ma-0"
                @change="onFilterChange()"
              />
            </div>
          </v-card-title>
          <v-card-text class="pa-0">
            <v-data-table
              :loading="isLoading"
              :headers="txTable"
              :items="transactions"
              :sort-by="['timestamp']"
              :sort-desc="[true]"
              :items-per-page="10"
              :footer-props="{'items-per-page-options': [10, 25, 50]}"
              no-data-text="this block does not contain any transactions"
              class="secondary--text"
              :server-items-length="total"
              :options.sync="options"
            >
              <template v-slot:header.label="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:header.type="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:header.id="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:header.sender="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:header.recipient="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:header.amount="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:header.fee="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:header.timestamp="{ header }">
                <span class="overline grey--text">{{ header.text }}</span>
              </template>

              <template v-slot:item.label="{ item }">
                <v-btn
                  :color="color(item.label)"
                  outlined
                  dark
                  small
                  style="width:30px;"
                  class="text-center"
                >
                  {{ item.label }}
                </v-btn>
              </template>

              <template v-slot:item.type="{ item }">
                {{ name(item.type) }}
              </template>

              <template v-slot:item.id="{ item }">
                <nuxt-link :to="{ path: '/transaction/' + item.id }" class="d-inline-block primary--text">
                  {{ item.id | truncateString }}
                </nuxt-link>
              </template>

              <template v-slot:item.sender="{ item }">
                <nuxt-link :to="{ path: '/address/' + item.sender }" class="d-inline-block primary--text">
                  {{ item.sender | truncateString }}
                </nuxt-link>
              </template>

              <template v-slot:item.recipient="{ item }">
                <nuxt-link
                  v-if="item.recipient"
                  :to="{ path: '/address/' + item.recipient }"
                  class="d-inline-block primary--text"
                >
                  {{ item.recipient | truncateString }}
                </nuxt-link>
                <span v-if="!item.recipient">N/A</span>
              </template>

              <template v-slot:item.amount="{ item }">
                {{ item.amount | parseAtomic | parseNumber }}
              </template>

              <template v-slot:item.fee="{ item }">
                {{ item.fee | parseAtomic | parseNumber }}
              </template>

              <template v-slot:item.timestamp="{ item }">
                {{ item.timestamp | parseTime }}
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
  import Vue from 'vue'
  import { Component, Watch } from 'vue-property-decorator'
  import '@nuxtjs/axios'

  @Component({
    components: {},
    computed: {},
    validate({ params }) {
      // return !isNaN(params.address)
      return true
    },
    async asyncData({ $axios, params }) {
      // Pagination still needs to be implemented possible that a store
      // may be required due to many transactions linked to active
      // addresses

      // eg: https://codepen.io/paulpv/pen/zWPKao

      // required: method to get total sum of transactions linked to the
      // an address in order to determine the `limit` parameter.

      const balance = await $axios.$get(process.env.LB_API + '/addresses/balance/details/' + params.address, {
        timeout: Number(process.env.AXIOS_TIMEOUT)
      })

      return {
        balance
      }
    }
  })

  export default class Address extends Vue {
    address = (this as any).$nuxt.$route.params.address
    copied = false
    balance = null
    transactions = []
    isLoading = false
    total = 0
    options: any = {}
    limit = 10

    async makeRequest() {
      this.isLoading = true
      try {
        const { data, headers } = await this.$axios.get(`${process.env.LB_API}/index/transactions/addresses/${this.address}?${this.queryParameters}`, {
          timeout: Number(process.env.AXIOS_TIMEOUT),
          headers: {
            'Access-Control-Expose-Headers': '*'
          }
        })
        this.total = Number(headers['x-total'])
        this.transactions = data
        if (this.transactions.length > 0) {
          this.transactions.forEach((tx: any) => {
            if (tx.sender === this.address) {
              tx.label = 'out'
            } else if (tx.recipient === this.address) {
              tx.label = 'in'
            } else {
              tx.label = 'out'
            }
          })
        }
      } catch (e) {
        // Do something
      } finally {
        this.isLoading = false
      }
    }

    get queryParameters() {
      return `offset=${this.offset}&limit=${this.options.itemsPerPage}&type=${this.filterType}`
    }

    get offset() {
      return (this.limit * this.options.page) - this.limit
    }

    get filterType() {
      return this.typeMap[this.txType]
    }

    onFilterChange() {
      this.makeRequest()
    }

    @Watch('options')
    onOptionsChange() {
      this.makeRequest()
    }

    txTable = [
      {
        text: 'I / O',
        align: 'center',
        value: 'label'
      },
      {
        text: 'Type',
        align: 'left',
        value: 'type'
      },
      {
        text: 'ID',
        align: 'left',
        value: 'id'
      },
      {
        text: 'Sender',
        align: 'left',
        value: 'sender'
      },
      {
        text: 'Recipient',
        align: 'left',
        value: 'recipient'
      },
      {
        text: 'Amount',
        align: 'right',
        value: 'amount'
      },
      {
        text: 'Fee',
        align: 'right',
        value: 'fee'
      },
      {
        text: 'Timestamp',
        align: 'center',
        value: 'timestamp'
      }
    ]

    txFilter = [
      { text: 'Anchor', value: 15 },
      { text: 'Transfer', value: 4 },
      { text: 'Start Lease', value: 8 },
      { text: 'Cancel Lease', value: 9 },
      { text: 'Mass Transfer', value: 11 }
    ]

    txType = 4

    typeMap: any = {
      1: 'genesis',
      4: 'transfer',
      8: 'start_lease',
      9: 'cancel_lease',
      11: 'mass_transfer',
      13: 'script',
      15: 'anchor',
      16: 'invoke_association',
      17: 'revoke_association',
      18: 'sponsor',
      19: 'cancel_sponsor'
    }

    name(value: number): string {
      if (value === 1) {
        return 'Genesis'
      } else if (value === 4) {
        return 'Transfer'
      } else if (value === 8) {
        return 'Lease'
      } else if (value === 9) {
        return 'Cancel Lease'
      } else if (value === 11) {
        return 'Mass Transfer'
      } else if (value === 13) {
        return 'Script'
      } else if (value === 15) {
        return 'Anchor'
      } else if (value === 16) {
        return 'Invoke Association'
      } else if (value === 17) {
        return 'Revoke Association'
      } else if (value === 18) {
        return 'Sponsor'
      } else if (value === 19) {
        return 'Cancel Sponsor'
      } else {
        return 'Unknown'
      }
    }

    color(value: string): string {
      if (value === 'out') {
        return 'orange'
      } else if (value === 'in') {
        return 'green'
      } else {
        return ''
      }
    }

    icon(value: number): string {
      // Genesis Transfer
      if (value === 1) {
        return 'mdi-power'
      } else if (value === 4) {
        // Transfer
        return 'mdi-send'
      } else if (value === 8) {
        // Lease
        return 'mdi-file-document-box-plus'
      } else if (value === 9) {
        // Cancel Lease
        return 'mdi-file-document-box-remove'
      } else if (value === 11) {
        // Mass Transfer
        return 'mdi-coins'
      } else if (value === 13) {
        // Set Script
        return 'mdi-script-text'
      } else if (value === 15) {
        // Anchor
        return 'mdi-anchor'
      } else if (value === 16) {
        // Invoke Association
        return 'mdi-link-plus'
      } else if (value === 17) {
        // Revoke Association
        return 'mdi-link-off'
      } else if (value === 18) {
        // Sponsor
        return 'mdi-heart'
      } else if (value === 19) {
        // Cancel Sponsor
        return 'mdi-heart-broken'
      } else {
        return 'Unknown'
      }
    }

    copy(): void {
      this.copied = true
    }
  }

</script>
