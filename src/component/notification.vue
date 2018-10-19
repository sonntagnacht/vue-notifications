<template>
  <transition name="fade">
    <div class="notification" :style="style">
      <b-alert
          :show="internalCountdown"
          :variant="variant"
          @dismissed="onDismissed"
          @dismiss-count-down="onCountDown"
          fade
          dismissible>
        <div class="message" v-html="message"></div>
        <b-progress
            class="linear-transition"
            v-if="progress.show"
            :max="progress.max"
            :variant="progress.variant"
            :value="internalCountdownProgress"
            height="4px">
        </b-progress>
      </b-alert>
    </div>
  </transition>
</template>

<script>

  import {getOffset} from '../../util/helper';

  //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
  //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
  //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

  export default {
    name    : 'notification',
    props   : {
      id        : String,
      variant   : String,
      show      : Boolean,
      message   : String,
      countdown : {
        type    : Number,
        default : 10
      },
      progress  : {
        type    : Object,
        default : function() {
          return {
            variant : String,
            show    : Boolean,
            max     : Number
          }
        }
      }
    },
    data    : function() {
      return {
        internalCountdown         : this.countdown,
        internalCountdownProgress : this.countdown,

        style : {
          // position : 'relative',
          // top      : '',
          // width    : 'auto',
          // height   : 'auto'
        }
      }
    },
    methods : {
      onCountDown(countdown) {
        this.internalCountdown         = countdown;
        this.internalCountdownProgress = countdown - 1;

        if (countdown === 0) {
          this.$emit('dismissed', {
            id : this.id
          });
        }
      },
      onDismissed() {
        this.internalCountdown = 0;
        //todo why is onDismissed not working???
        this.$emit('dismissed', {
          id : this.id
        });
      }
    },
    mounted : function() {
      // set default values for progress if not set
      for (let i in this.progress) {
        if (this.progress.hasOwnProperty(i) && null === this.progress[i]) {
          switch (i) {
            case 'variant':
              this.progress[i] = this.variant;
              break;
            case 'max':
              this.progress[i] = this.countdown;
              break;
          }
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  // why is [scoped] not working here?
  .notification {
    transition: all 1s;
    .progress {
      margin-top: 5px;
    }

    .message {
      font-size: 14px;
    }

    /deep/ .alert {
      padding: 0.25rem 1.75rem 0.5rem 0.5rem;
      .close {
        padding: 0.2rem 0.5rem 0.25rem 0.5rem;
      }
    }

    &.fade-enter {
      opacity: 0;
      transform: translateY(25px);
    }

    &.fade-leave-to {
      opacity: 0;
      transform: translateY(-25px);
    }

    &.fade-move {
      transition: transform 1s;
    }

    &.fade-leave,
    &.fade-leave-active {
      position: absolute;
    }
  }
</style>