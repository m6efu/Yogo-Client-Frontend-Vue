<template>
  <div
    v-if="incomingCall"
    class="callalert__wrapper"
    :style="styles.incomingCallWrapper"
  >
    <div :style="styles.callContainer">
      <div :style="styles.headerWrapper">
        <div :style="styles.callDetail">
          <div :style="styles.name">
            {{ incomingCall.sender.name }}
          </div>
          <div :style="styles.callType">
            <template v-if="incomingCall.type === 'video'">
              <img :src="icons.video" :alt="STRINGS.INCOMING_VIDEO_CALL" />
              <span :style="styles.callType.span">
                {{ STRINGS.INCOMING_VIDEO_CALL }}
              </span>
            </template>
            <template v-else>
              <img :src="icons.audio" :alt="STRINGS.INCOMING_AUDIO_CALL" />
              <span :style="styles.callType.span">
                {{ STRINGS.INCOMING_AUDIO_CALL }}
              </span>
            </template>
          </div>
        </div>
        <div :style="styles.thumbnail">
          <comet-chat-avatar
            corner-radius="50%"
            :image="incomingCall.sender.avatar"
          />
        </div>
      </div>
      <div :style="styles.headerButton">
        <button :style="styles.declineButton" @click="rejectCall">
          {{ STRINGS.DECLINE }}
        </button>
        <button :style="styles.acceptButton" @click="acceptCall">
          {{ STRINGS.ACCEPT }}
        </button>
      </div>
    </div>
  </div>
</template>
<script>
import { CometChat } from "@cometchat-pro/chat";

import {
  STRING_MESSAGES,
  DEFAULT_OBJECT_PROP,
} from "../../../resources/constants";

import { propertyCheck, cometChatCommon } from "../../../mixins/";

import { CometChatManager } from "../../../util/controller";
import { SvgAvatar } from "../../../util/svgavatar";
import * as enums from "../../../util/enums.js";

import { CometChatAvatar } from "../../Shared/";

import { CallAlertManager } from "./controller";

import * as style from "./style";

import audioCallIcon from "./resources/incomingaudiocall.png";
import videoCallIcon from "./resources/incomingvideocall.png";

import { incomingCallAlert } from "../../../resources/audio/";

let incomingAlert;
let callAlertManager;

export default {
  name: "CometChatIncomingCall",
  mixins: [propertyCheck, cometChatCommon],
  components: { CometChatAvatar },
  props: {
    theme: { ...DEFAULT_OBJECT_PROP },
    callInProgress: { ...DEFAULT_OBJECT_PROP },
  },
  data() {
    return {
      incomingCall: null,
    };
  },
  computed: {
    styles() {
      return {
        incomingCallWrapper: style.incomingCallWrapperStyle(
          this.theme,
          {}, // widgetsettings opted out
          this.hasProperty
        ),
        name: style.nameStyle(),
        callType: style.callTypeStyle(),
        thumbnail: style.thumbnailStyle(),
        callDetail: style.callDetailStyle(),
        headerButton: style.headerButtonStyle(),
        callContainer: style.callContainerStyle(),
        headerWrapper: style.headerWrapperStyle(),
        acceptButton: style.buttonStyle(this.theme, true),
        declineButton: style.buttonStyle(this.theme, false),
      };
    },
    icons() {
      return {
        audio: audioCallIcon,
        video: videoCallIcon,
      };
    },
    STRINGS() {
      return STRING_MESSAGES;
    },
  },
  methods: {
    playIncomingAlert() {
      incomingAlert.currentTime = 0;
      if (typeof incomingAlert.loop == "boolean") {
        incomingAlert.loop = true;
      } else {
        incomingAlert.addEventListener(
          "ended",
          function () {
            this.currentTime = 0;
            this.play();
          },
          false
        );
      }
      incomingAlert.play();
    },

    pauseIncomingAlert() {
      incomingAlert.pause();
    },
    markMessageAsRead(message) {
      const receiverType = message.receiverType;
      const receiverId =
        receiverType === "user" ? message.sender.uid : message.receiverId;

      if (this.hasProperty(message, "readAt") === false) {
        CometChat.markAsRead(message.id, receiverId, receiverType);
      }
    },
    async incomingCallReceived(incomingCall) {
      const activeCall = CometChat.getActiveCall();

      if (activeCall) {
        try {
          const rejectedCall = await CometChat.rejectCall(
            incomingCall.sessionId,
            CometChat.CALL_STATUS.BUSY
          );

          this.markMessageAsRead(incomingCall);
          this.emitAction("rejectedIncomingCall", {
            incomingCall,
            rejectedCall,
          });
        } catch (error) {
          this.emitAction("callError", { error });
          console.log("Call rejection failed with error:", error);
        }
      } else if (this.incomingCall === null) {
        this.playIncomingAlert();

        if (incomingCall.sender.avatar === false) {
          const uid = incomingCall.sender.uid;
          const char = incomingCall.sender.name.charAt(0).toUpperCase();

          incomingCall.sender.avatar = SvgAvatar.getAvatar(uid, char);
        }

        this.incomingCall = incomingCall;
      }
    },
    incomingCallCancelled() {
      this.pauseIncomingAlert();
      this.incomingCall = null;
    },
    async rejectCall() {
      this.pauseIncomingAlert();

      try {
        const rejectedCall = await CometChatManager.rejectCall(
          this.incomingCall.sessionId,
          CometChat.CALL_STATUS.REJECTED
        );

        this.emitAction("rejectedIncomingCall", {
          incomingCall: this.incomingCall,
          rejectedCall,
        });
      } catch (error) {
        this.emitAction("callError", { error });
      } finally {
        this.incomingCall = null;
      }
    },
    acceptCall() {
      this.pauseIncomingAlert();
      this.emitAction("acceptIncomingCall", {
        incomingCall: this.incomingCall,
      });

      setTimeout(() => {
        this.incomingCall = null;
      }, 100);
    },
    callScreenUpdateHandler(key, call) {
      switch (key) {
        case enums.INCOMING_CALL_RECEIVED:
          this.incomingCallReceived(call);
          break;
        case enums.INCOMING_CALL_CANCELLED:
          this.incomingCallCancelled();
          break;
        default:
          break;
      }
    },
  },
  beforeMount() {
    incomingAlert = new Audio(incomingCallAlert);

    callAlertManager = new CallAlertManager();
    callAlertManager.attachListeners(this.callScreenUpdateHandler);
  },
  beforeDestroy() {
    if (callAlertManager) {
      callAlertManager.removeListeners();
      callAlertManager = null;
    }
  },
};
</script>

<style scoped>
.callalert__wrapper {
  animation: slideDown 250ms ease;
}
.callalert__wrapper * {
  box-sizing: border-box !important;
  font-family: var(--call-alert-font-family) !important;
}
@keyframes slideDown {
  0% {
    transform: translateY(-50px);
  }
  100% {
    transform: translateY(0px);
  }
}
</style>