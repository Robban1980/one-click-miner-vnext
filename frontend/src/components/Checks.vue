<template>
  <div class="container">
    <div v-if="prerequisiteInstall" class="col-286">
      <p>{{ $t("checks.prerequisite") }}</p>
    </div>
    <div v-if="!prerequisiteInstall && checkStatus !== 'Failed'" class="col-286">
      <p >{{checkStatus}}...</p>
    </div>
    <div v-if="!prerequisiteInstall && checkStatus === 'Failed'" class="col-wide">
      <div class="failureReason" v-if="checkStatus === 'Failed'">
          {{ $t("checks.checks_failed") }}:<br/>
          {{failureReason}}
      </div>
      <p v-if="!prerequisiteInstall && checkStatus === 'Failed'">
        <a class="button" @click="check">{{ $t('generic.retry') }}</a>
      </p>
    </div>
  </div>
  
</template>

<script>


export default {
  data() {
    return {
      prerequisiteInstall: false,
      checkStatus: this.$t("checks.checking_mining_software"),
      failureReason: ""
    };
  },
  mounted() {
    this.check();
    var self = this;
    wails.Events.On("checkStatus",(result) => {
      if (result === "Failed") {
        self.checkStatus = result;
      } else {
        self.checkStatus = this.$t("checks." + result);
      }
	  });
    wails.Events.On("prerequisiteInstall",(result) => {
      self.prerequisiteInstall = (result === "1");
    });
  },
  methods: {
    check: function() {
  	  var self = this;
	  
      window.backend.Backend.PerformChecks().then(result => {
          if(result === "ok") {
      			self.startMining()
		      } else {
            self.failureReason = result;
          }
      });
    },
    startMining: function() {
      var self = this;
      window.backend.Backend.StartMining().then(result => {
        self.$emit('mining');
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div.failureReason {
  height: 200px;
  overflow-y: auto;
  font-family: 'Courier New', Courier, monospace;
  color: red;
  border: 1px solid red;
  width: 600px;
  margin: 0 auto;
}
</style>
