<template>
  <div :style="styles.container">
    <div :style="styles.wrapper">
      <div :style="styles.thumbnail" v-if="isGroup">
        <comet-chat-avatar
          border-width="1px"
          corner-radius="50%"
          :image="parsedMessage.sender.avatar"
          :border-color="theme.borderColor.primary"
        />
      </div>
      <div :style="styles.detail">
        <div :style="styles.nameWrapper" v-if="isGroup">
          <span :style="styles.name">{{ parsedMessage.sender.name }} </span>
        </div>
        <div
          ref="messageBubbleWrapperRef"
          :style="styles.messageActionWrapper"
          class="receiver__file__message__action__wrapper"
        >
          <comet-chat-message-actions
            :is-group="isGroup"
            v-bind="commonProps"
            @action="actionHandler"
          />
          <div :style="styles.fileContainer">
            <div :style="styles.fileWrapper">
              <a
                target="_blank"
                rel="noopener noreferrer"
                :style="styles.fileLink"
                class="receiver__message__file__link"
                :href="parsedMessage.data.attachments[0].url"
                ><span :style="styles.fileLinkContainer">
                  <span>
                    {{ parsedMessage.data.attachments[0].name }}
                  </span>
                  <img
                    alt="file"
                    :src="fileIcon"
                    class="receiver__message__file__img"
                  />
                </span>
              </a>
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
          <comet-chat-read-reciept :theme="theme" :message="parsedMessage" />
          <comet-chat-threaded-message-reply-count
            v-bind="commonProps"
            v-if="!parentMessageId"
            @action="actionHandler"
          />
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import {
  DEFAULT_OBJECT_PROP,
  DEFAULT_STRING_PROP,
} from "../../../resources/constants";

import { cometChatCommon, cometChatBubbles } from "../../../mixins/";

import { CometChatAvatar } from "../../Shared";
import CometChatReadReciept from "../CometChatReadReciept/CometChatReadReciept";
import CometChatMessageActions from "../CometChatMessageActions/CometChatMessageActions";
import CometChatMessageReactions from "../Extensions/CometChatMessageReactions/CometChatMessageReactions";
import CometChatThreadedMessageReplyCount from "../CometChatThreadedMessageReplyCount/CometChatThreadedMessageReplyCount";

import * as style from "./style";

import blueFile from "./resources/file-blue.svg";

export default {
  name: "CometChatReceiverFileMessageBubble",
  mixins: [cometChatCommon, cometChatBubbles],
  components: {
    CometChatAvatar,
    CometChatReadReciept,
    CometChatMessageActions,
    CometChatMessageReactions,
    CometChatThreadedMessageReplyCount,
  },
  props: {
    theme: { ...DEFAULT_OBJECT_PROP },
    message: { ...DEFAULT_OBJECT_PROP },
    loggedInUser: { ...DEFAULT_OBJECT_PROP },
    widgetconfig: { ...DEFAULT_OBJECT_PROP },
    parentMessageId: { ...DEFAULT_STRING_PROP },
  },
  data() {
    return {
      messageFrom: "receiver",
    };
  },
  computed: {
    styles() {
      return {
        name: style.nameStyle(this.theme),
        detail: style.messageDetailStyle(),
        wrapper: style.messageWrapperStyle(),
        thumbnail: style.messageThumbnailStyle(),
        container: style.messageContainerStyle(),
        infoWrapper: style.messageInfoWrapperStyle(),
        fileLink: style.messageFileLinkStyle(this.theme),
        fileContainer: style.messageFileContainerStyle(),
        nameWrapper: style.nameWrapperStyle(this.isGroup),
        timestamp: style.messageTimestampStyle(this.theme),
        fileWrapper: style.messageFileWrapperStyle(this.theme),
        fileLinkContainer: style.messageFileLinkContainerStyle(),
        messageReactionsWrapper: style.messageReactionsWrapperStyle(),
        messageActionWrapper: style.messageActionWrapperStyle(
          this.parentMessageId
        ),
      };
    },
    fileIcon() {
      return blueFile;
    },
  },
};
</script>
<style scoped>
.receiver__message__file__link {
  background-color: var(--file-recieve-message-bg-color) !important;
}
.receiver__message__file__link:visited,
.receiver__message__file__link:active,
.receiver__message__file__link:hover {
  text-decoration: none;
  color: var(--file-recieve-message-color);
}
.receiver__message__file__img {
  margin-left: 8px;
  border-radius: 4px;
  background-color: var(--file-recieve-message-bg-color);
}
</style>
<style>
.receiver__file__message__action__wrapper:hover ul.message__actions {
  visibility: var(
    --receiver-file-message-bubble-hover-display,
    visible
  ) !important;
}
</style>