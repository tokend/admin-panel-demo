<template>
  <div class="request-list">
    <div class="request-list__filters-wrp">
      <div class="app-list-filters">
        <select-field
          class="app-list-filters__field"
          v-model="filters.state"
          :label="'kyc-recovery-request-list.lbl-state' | globalize"
        >
          <option
            v-for="(item, s) in KYC_REQUEST_STATES"
            :key="`kyc-request-${s}`"
            :value="item.stateI"
          >
            {{ item.stateI | globalizeRequestStateI }}
          </option>
        </select-field>

        <input-field
          class="app-list-filters__field"
          v-model.trim="filters.requestor"
          :label="'kyc-recovery-request-list.lbl-requestor' | globalize"
          autocomplete-type="email"
        />
      </div>
    </div>

    <div class="request-list__list-wrp">
      <template v-if="list && list.length">
        <div class="app-list">
          <div class="app-list__header">
            <span class="app-list__cell">
              {{ "kyc-recovery-request-list.mail" | globalize }}
            </span>
            <span class="app-list__cell app-list__cell--right">
              {{ "kyc-recovery-request-list.state" | globalize }}
            </span>
            <span class="app-list__cell app-list__cell--right">
              {{ "kyc-recovery-request-list.last-upd" | globalize }}
            </span>
          </div>
          <button
            class="app-list__li"
            v-for="item in list"
            :key="item.id"
            @click="showReqestDetails(item.requestor.id)"
          >
            <email-getter
              class="app-list__cell app-list__cell--important"
              :account-id="item.requestor.id"
              is-titled
            />
            <span class="app-list__cell app-list__cell--right">
              {{ item.stateI | globalizeRequestStateI }}
            </span>
            <span class="app-list__cell app-list__cell--right">
              {{ item.updatedAt | formatDate }}
            </span>
          </button>
        </div>
      </template>

      <template v-else>
        <div class="app-list">
          <template v-if="isLoading">
            <p class="app-list__li">
              <span class="app-list__cell app-list__cell--center">
                {{ "kyc-recovery-request-list.loading" | globalize }}
              </span>
            </p>
          </template>
          <template v-else>
            <p class="app-list__li">
              <span class="app-list__cell app-list__cell--center">
                {{ "kyc-recovery-request-list.fail-loading" | globalize }}
              </span>
            </p>
          </template>
        </div>
      </template>

      <div class="app__more-btn-wrp">
        <collection-loader
          :first-page-loader="loadList"
          @first-page-load="setList"
          @next-page-load="extendList"
          ref="collectionLoaderBtn"
        />
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'

import InputField from '@comcom/fields/InputField'
import SelectField from '@comcom/fields/SelectField'

import { EmailGetter } from '@comcom/getters'

import { clearObject } from '@/utils/clearObject'

import { KYC_REQUEST_STATES } from '@/constants'

import { api } from '@/api'
import apiHelper from '@/apiHelper'

import { ErrorHandler } from '@/utils/ErrorHandler'
import { CollectionLoader } from '@/components/common'
import { base } from '@tokend/js-sdk'

export default {
  name: 'kyc-recovery-request-list',
  components: {
    EmailGetter,
    SelectField,
    InputField,
    CollectionLoader,
  },

  data () {
    return {
      isLoading: false,
      list: [],
      filters: {
        state: KYC_REQUEST_STATES.pending.stateI,
        requestor: '',
        type: '',
        pendingTasks: '',
      },
      KYC_REQUEST_STATES,
    }
  },

  watch: {
    'filters.state' () { this.reloadCollectionLoader() },
    'filters.pendingTasks' () { this.reloadCollectionLoader() },
    'filters.requestor': _.throttle(function () {
      this.reloadCollectionLoader()
    }, 1000),
  },

  methods: {
    async loadList () {
      this.isLoading = true
      let response = {}
      try {
        const requestor =
          await this.getRequestorAccountId(this.filters.requestor)
        response = await api.getWithSignature('/v3/kyc_recovery_requests', {
          page: { order: 'desc' },
          filter: clearObject({
            pending_tasks: this.filters.pendingTasks,
            state: this.filters.state,
            requestor: requestor,
          }),
          include: ['request_details.target_account'],
        })
      } catch (error) {
        ErrorHandler.processWithoutFeedback(error)
      }
      this.isLoading = false
      return response
    },

    async getRequestorAccountId (requestor) {
      if (base.Keypair.isValidPublicKey(requestor)) {
        return requestor
      } else {
        try {
          const address = await apiHelper.users.getAccountIdByEmail(requestor)
          return address || requestor
        } catch (error) {
          return requestor
        }
      }
    },

    setList (data) {
      this.list = data
    },
    async extendList (data) {
      this.list = this.list.concat(data)
    },

    showReqestDetails (id) {
      if (id) {
        this.$router.push({
          name: 'kycRecoveryRequests.show',
          params: {
            id: id,
          },
        })
      }
    },

    reloadCollectionLoader () {
      this.$refs.collectionLoaderBtn.loadFirstPage()
    },
  },
}
</script>

<style scoped>
.request-list__filters-wrp {
  margin-bottom: 4rem;
}
</style>
