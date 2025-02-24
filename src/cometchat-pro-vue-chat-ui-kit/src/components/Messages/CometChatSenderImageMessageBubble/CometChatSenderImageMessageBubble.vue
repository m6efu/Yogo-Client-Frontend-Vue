<template>
  <div :style="styles.container">
    <div
      ref="messageBubbleWrapperRef"
      :style="styles.messageActionWrapper"
      class="sender__image__message__action__wrapper"
    >
      <comet-chat-message-actions
        v-bind="commonProps"
        @action="actionHandler"
      />
      <div :style="styles.wrapper">
        <div
          :style="styles.imgWrapper"
          class="message__img__wrapper"
          @click="emitEvent('action', { action: 'viewActualImage', message })"
        >
          <img
            alt="message"
            :src="imageUrl"
            :style="styles.img"
            @load="positionTooltip"
          />
        </div>
      </div>
    </div>
    <comet-chat-message-reactions
      ref="reactionRef"
      v-bind="commonProps"
      :message-from="messageFrom"
      :logged-in-user="loggedInUser"
    />
    <div :style="styles.infoWrapper">
      <comet-chat-threaded-message-reply-count
        v-bind="commonProps"
        v-if="!parentMessageId"
        @action="actionHandler"
      />
      <comet-chat-read-reciept :theme="theme" :message="parsedMessage" />
    </div>
  </div>
</template>
<script>
import {
  DEFAULT_OBJECT_PROP,
  DEFAULT_STRING_PROP,
} from "../../../resources/constants";

import {
  cometChatCommon,
  cometChatBubbles,
  thumbnailGeneration,
} from "../../../mixins/";

import CometChatReadReciept from "../CometChatReadReciept/CometChatReadReciept";
import CometChatMessageActions from "../CometChatMessageActions/CometChatMessageActions";
import CometChatMessageReactions from "../Extensions/CometChatMessageReactions/CometChatMessageReactions";
import CometChatThreadedMessageReplyCount from "../CometChatThreadedMessageReplyCount/CometChatThreadedMessageReplyCount";

import * as style from "./style";

import srcIcon from "./resources/1px.png";

export default {
  name: "CometChatSenderImageMessageBubble",
  mixins: [cometChatCommon, cometChatBubbles, thumbnailGeneration],
  components: {
    CometChatReadReciept,
    CometChatMessageActions,
    CometChatMessageReactions,
    CometChatThreadedMessageReplyCount,
  },
  props: {
    item: { ...DEFAULT_OBJECT_PROP },
    theme: { ...DEFAULT_OBJECT_PROP },
    message: { ...DEFAULT_OBJECT_PROP },
    loggedInUser: { ...DEFAULT_OBJECT_PROP },
    widgetconfig: { ...DEFAULT_OBJECT_PROP },
    parentMessageId: { ...DEFAULT_STRING_PROP },
  },
  data() {
    return {
      imageUrl: srcIcon,
      windowWidth: 1000,
      messageFrom: "sender",
    };
  },
  computed: {
    styles() {
      return {
        img: style.messageImg(),
        wrapper: style.messageWrapperStyle(),
        imgWrapper: style.messageImgWrapper(),
        container: style.messageContainerStyle(),
        infoWrapper: style.messageInfoWrapperStyle(),
        messageReactionsWrapper: style.messageReactionsWrapperStyle(),
        messageActionWrapper: style.messageActionWrapperStyle(
          this.parentMessageId
        ),
      };
    },
  },
  mounted() {
    this.onResize();
    this.setImage();
    window.addEventListener("resize", this.onResize);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.onResize);
  },
};
</script>
<style scoped>
@media (min-width: 320px) and (max-width: 767px) {
  .message__img__wrapper {
    height: 100px !important;
    min-width: 50px !important;
    max-width: 150px !important;
    padding: 2px 2px !important;
  }
}
</style>
<style>
.sender__image__message__action__wrapper:hover ul.message__actions {
  visibility: var(
    --sender-image-message-bubble-hover-display,
    visible
  ) !important;
}
</style>