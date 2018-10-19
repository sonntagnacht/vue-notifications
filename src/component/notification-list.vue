<template>
  <transition name="fade-list">
    <div id="notification-list" v-if="list.length > 0">
      <notification v-for="(item, key, index) in list" :key="item.id"
                    :id="item.id"
                    :variant="item.variant"
                    :message="item.message"
                    :countdown="item.countdown"
                    :progress="item.progress"
                    :show="item.show"
                    @dismissed="onNotificationDismissed"
      ></notification>
    </div>
  </transition>
</template>

<script>

  import Vue from 'vue';
  import {
    merge as _merge,
    forEach as _forEach,
    isString as _isString,
    uniqueId as _uniqueId,
    findIndex as _findIndex,
    isFunction as _isFunction
  } from 'lodash-es';
  import notification from './notification.vue'

  //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
  //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
  //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

  let model = {
    variant     : 'info',
    show        : true,
    message     : '',
    countdown   : 10,
    type        : null,
    progress    : {
      variant : null,
      show    : false,
      max     : null
    },
    onDismissed : () => {
    }
  };

  const notificationList = {
    name          : 'notification-list',
    components    : {
      notification
    },
    data() {
      return {
        list : []
      }
    },
    methods       : {
      /**
       * @description data has to be a object like {message string, variant [danger,info,warning,success], countdown? int}
       *
       * @param {Object} data - the options of the next notification
       * @param {String} data.variant - the style of the notification [danger, info, warning, success]
       * @param {Boolean} data.show - whether to show the Notification or not (default: true)
       * @param {String} data.type - (optional) type of the message. can be used to show a message of a type only once
       * @param {String} data.message - the message to show, can be HTML
       * @param {Number} data.countdown - the time this message will be shown until it disappears
       * @param {Function} data.onDismissed - callback function which is executed when the notification disappears
       * @param {Object} data.progress - the options for a progress bar
       * @param {String} data.progress.variant - the style of the progress bar [danger, info, warning, success]. (default: inherits from notification variant)
       * @param {Boolean} data.progress.show - whether to show the progress bar or not (default: false)
       * @param {Number} data.progress.max - the maximum count for the progress bar (default: inherits from notification countdown)
       */
      addNotification         : function(data) {
        let alert = _merge({}, model, data);

        if (this.hasNotificationOfType(alert.type)) {
          return;
        }

        alert.id = _uniqueId('n-');
        this.list.push(alert);
      },
      onNotificationDismissed : function(notification) {
        let idx = _findIndex(this.list, function(n) {
          if (n.id === notification.id) {
            if (_isFunction(n.onDismissed)) {
              n.onDismissed(n);
            }
            return true;
          }

          return false;
        });
        this.list.splice(idx, 1);
      },
      hasNotificationOfType   : function(type) {
        if (false === _isString(type)) {
          return false;
        }
        if (false === notificationType.hasOwnProperty(type)) {
          console.warn('Unknown notificationType: %o', type);
        }
        let hasOneOfType = false;
        _forEach(this.list, (notification, idx) => {
          if (_isString(notification.type) && notification.type === type) {
            hasOneOfType = true;
          }
        });

        return hasOneOfType;
      }
    },
    mounted       : function() {
      this.$root.$on('notification', this.addNotification);
      Vue.prototype.$notification = {
        add : this.addNotification
      };
    },
    beforeDestroy : function() {
      this.$root.$off('notification');
      Vue.prototype.$notification = null;
    }
  };

  const notificationType = {
    ERROR_MAP_NO_STATION        : 'ERROR_MAP_NO_STATION',
    ERROR_MAP_NO_RENTAL_OBJECTS : 'ERROR_MAP_NO_RENTAL_OBJECTS',
    ERROR_RENTAL_OBJECT_DETAIL  : 'ERROR_RENTAL_OBJECT_DETAIL',
    ERROR_BOOKING_ENDED_FAILED  : 'ERROR_BOOKING_ENDED_FAILED',
    MAP_TIMEZONE_MISSMATCH      : 'MAP_TIMEZONE_MISSMATCH'
  };

  export default notificationList;
  export {notificationType};

</script>

<style lang="scss" scoped>
  #notification-list {
    position: fixed;
    top: 50px;
    right: 5%;
    bottom: 0;
    min-width: 10%;
    max-width: 90%;
    z-index: 1000;
    pointer-events: none;
    transition: all 1s;

    & > div {
      pointer-events: all;
      overflow-y: scroll;
      max-height: 100%;
      padding-right: 17px;
      margin-right: -17px;
      padding-top: 17px;
    }

    &.fade-list-enter {
      opacity: 0;
      transform: translateY(25px);
    }

    &.fade-list-leave-to {
      opacity: 0;
      transform: translateY(-25px);
    }

  }
</style>