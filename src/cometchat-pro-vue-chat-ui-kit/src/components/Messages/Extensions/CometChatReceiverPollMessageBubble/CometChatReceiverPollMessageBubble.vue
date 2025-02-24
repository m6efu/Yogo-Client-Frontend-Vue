<template>
  <div :style="styles.messageContainer">
    <div :style="styles.messageWrapper">
      <div v-if="isGroup" :style="styles.messageThumbnail">
        <comet-chat-avatar
          border-width="1px"
          corner-radius="50%"
          :border-color="theme.borderColor.primary"
          :image="parsedMessage.sender.avatar"
        />
      </div>
      <div :style="styles.messageDetail">
        <div v-if="isGroup" :style="styles.nameWrapper">
          <span :style="styles.name">{{ message.sender.name }}</span>
        </div>

        <div
          ref="messageBubbleWrapperRef"
          :style="styles.messageActionWrapper"
          class="receiver__poll__message__action__wrapper"
        >
          <comet-chat-message-actions
            :is-group="isGroup"
            v-bind="commonProps"
            @action="actionHandler"
          />
          <div :style="styles.messageTextContainer">
            <div :style="styles.messageTextWrapper">
              <p :style="styles.pollQuestion">{{ pollData.question }}</p>
              <ul :style="styles.pollAnswer">
                <li
                  :key="i"
                  :style="styles.pollAnswer.li"
                  v-for="(pollOption, i) in pollOptions"
                  @click="answerPoll(i)"
                >
                  <div
                    :style="getPollPercentStyle(getPollPercent(pollOption))"
                  ></div>
                  <div :style="getAnswerWrapperStyle(pollOption)">
                    <span :style="getAnswerWrapperStyle(pollOption).span">
                      {{ getPollPercent(pollOption) }}
                    </span>
                    <p :style="getAnswerWrapperStyle(pollOption).p">
                      {{ pollOption.text }}
                    </p>
                  </div>
                </li>
              </ul>
              <p :style="styles.pollTotal">{{ totalText }}</p>
            </div>
          </div>
        </div>
        <comet-chat-message-reactions
          ref="reactionRef"
          v-bind="commonProps"
          :message-from="messageFrom"
          :logged-in-user="loggedInUser"
        />
        <div :style="styles.messageInfoWrapper">
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
import { CometChat } from "@cometchat-pro/chat";

import {
  DEFAULT_OBJECT_PROP,
  DEFAULT_STRING_PROP,
} from "../../../../resources/constants";

import { polls, cometChatCommon, cometChatBubbles } from "../../../../mixins/";

import { CometChatAvatar } from "../../../Shared";
import CometChatReadReciept from "../../CometChatReadReciept/CometChatReadReciept";
import CometChatMessageActions from "../../CometChatMessageActions/CometChatMessageActions";
import CometChatMessageReactions from "../CometChatMessageReactions/CometChatMessageReactions";
import CometChatThreadedMessageReplyCount from "../../CometChatThreadedMessageReplyCount/CometChatThreadedMessageReplyCount";

import * as style from "./style";

import checkIcon from "./resources/check.svg";

export default {
  name: "CometChatReceiverPollMessageBubble",
  mixins: [polls, cometChatCommon, cometChatBubbles],
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
        pollTotal: style.pollTotalStyle(),
        name: style.nameStyle(this.theme),
        pollQuestion: style.pollQuestionStyle(),
        messageDetail: style.messageDetailStyle(),
        messageWrapper: style.messageWrapperStyle(),
        pollAnswer: style.pollAnswerStyle(this.theme),
        messageThumbnail: style.messageThumbnailStyle(),
        messageContainer: style.messageContainerStyle(),
        nameWrapper: style.nameWrapperStyle(this.isGroup),
        messageInfoWrapper: style.messageInfoWrapperStyle(),
        messageTextContainer: style.messageTextContainerStyle(),
        messageTimestamp: style.messageTimestampStyle(this.theme),
        messageTextWrapper: style.messageTextWrapperStyle(this.theme),
        messageReactionsWrapper: style.messageReactionsWrapperStyle(),
        messageActionWrapper: style.messageActionWrapperStyle(
          this.parentMessageId
        ),
      };
    },
  },
  methods: {
    getPollPercent(pollOption) {
      let width = "0%";

      if (this.total) {
        width = (pollOption.count / this.total).toLocaleString("en", {
          style: "percent",
        });
      }
      return width;
    },
    getPollPercentStyle(percent) {
      return style.pollPercentStyle(this.theme, percent);
    },
    getAnswerWrapperStyle(pollOption) {
      return style.answerWrapperStyle(
        this.theme,
        this.loggedInUser,
        pollOption,
        checkIcon
      );
    },
    async answerPoll(selectedOption) {
      try {
        const message = await CometChat.callExtension(
          "polls",
          "POST",
          "v1/vote",
          {
            vote: selectedOption,
            id: this.pollId,
          }
        );

        this.emitAction("pollAnswered", { message });
      } catch (error) {
        console.log("answerPoll error", error);
      }
    },
  },
};
</script>
<style>
.receiver__poll__message__action__wrapper:hover ul.message__actions {
  visibility: var(
    --receiver-message-poll-bubble-hover-display,
    visible
  ) !important;
}
</style>