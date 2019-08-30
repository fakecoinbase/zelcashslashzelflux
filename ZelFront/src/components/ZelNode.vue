<template>
  <div class="zelnodeSection">
    <div v-if="zelnodeSection === 'getinfo'">
      <div class="status">
        <h4>
          ZelNode owner Zel ID: {{ userconfig.zelid }}
        </h4>
        <h4>
          Status: {{ getZelNodeStatusResponse.zelnodeStatus }}
        </h4>
      </div>

      <div class="getInfoResponse">
        <p>
          ZelCash version: {{ getInfoResponse.message.version }}
        </p>
        <p>
          Protocol version: {{ getInfoResponse.message.protocolversion }}
        </p>
        <p>
          Current Blockchain Height: {{ getInfoResponse.message.blocks }}
        </p>
        <div v-if="getInfoResponse.message.errors != ''">
          <p>
            Error: {{ getInfoResponse.message.errors }}
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vuex, { mapState } from 'vuex';
import Vue from 'vue';

import ZelCashService from '@/services/ZelCashService';

Vue.use(Vuex);

export default {
  name: 'ZelNode',
  data() {
    return {
      getInfoResponse: {
        status: '',
        message: '',
      },
      getZelNodeStatusResponse: {
        status: '',
        message: '',
        zelnodeStatus: 'Checking status...',
      },
    };
  },
  computed: {
    ...mapState([
      'config',
      'userconfig',
      'privilage',
      'loginPhrase',
      'zelnodeSection',
    ]),
  },
  watch: {
    zelnodeSection(val, oldVal) {
      console.log(val, oldVal);
      switch (val) {
        case 'getinfo':
          this.zelcashGetInfo();
          this.zelcashGetZelNodeStatus();
          break;
        case null:
          console.log('ZelNode Section hidden');
          break;
        default:
          console.log('ZelNode Section: Unrecognised method'); // should not be seeable if all works correctly
      }
    },
  },
  mounted() {
    switch (this.zelnodeSection) {
      case 'getinfo':
        this.zelcashGetInfo();
        this.zelcashGetZelNodeStatus();
        break;
      case null:
        console.log('ZelNode Section hidden');
        break;
      default:
        console.log('ZelNode Section: Unrecognised method');
    }
  },
  methods: {
    async zelcashGetInfo() {
      const response = await ZelCashService.getInfo();
      this.getInfoResponse.status = response.data.status;
      this.getInfoResponse.message = response.data.data;
    },
    async zelcashGetZelNodeStatus() {
      // TODO more code statuses?
      const response = await ZelCashService.getZelNodeStatus();
      this.getZelNodeStatusResponse.status = response.data.status;
      this.getZelNodeStatusResponse.message = response.data.data;
      console.log(this.getZelNodeStatusResponse.message);
      if (this.getZelNodeStatusResponse.message) {
        if (this.getZelNodeStatusResponse.message.status === 4) {
          this.getZelNodeStatusResponse.zelnodeStatus = 'ZelNode is working correctly';
        } else {
          const statusCode = this.getZelNodeStatusResponse.message.code || this.getZelNodeStatusResponse.message.status;
          this.getZelNodeStatusResponse.zelnodeStatus = `Error status code: ${statusCode}. ZelNode not yet active. Flux is running with limited capabilities.`;
        }
      }
    },
  },
};
</script>