<template>
  <div class="p-4 mb-2 mt-1 border-2 rounded-xl flex" :class="message.read ? 'bg-gray-100 dark:border-gray-700 dark:bg-gray-800' : 'bg-primary-100 dark:bg-gray-700 border-primary-200 dark:border-gray-600'">
    <input v-if="!admin" type="checkbox" class="mr-2 self-center" @change="select" ref="checkbox" />
    <div class="w-full">
      <div v-if="message.type == 'post_mention'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          mentioned you in
          <TextLink v-if="message.data.post" :to="`/posts/${message.data.post._id}`">their post</TextLink>
          <span v-else>a deleted post :(</span>
        </span>
        <Post class="mt-4" :post="message.data.post" v-if="message.data.post" />
      </div>
      <div v-else-if="message.type == 'follow'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          is now following you
        </span>
      </div>
      <div v-else-if="message.type == 'comment'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          commented on
          <TextLink v-if="message.data.comment && message.data.post" :to="`/posts/${message.data.post._id}#comments-${message.data.comment._id}`">your post</TextLink>
          <span v-else>one of your posts that is now deleted :(</span>
        </span>
        <Comment v-if="message.data.comment && message.data.post" class="pt-4" :comment="message.data.comment" :standalone="true" />
      </div>
      <div v-else-if="message.type == 'comment_reply'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          replied to
          <TextLink v-if="message.data.comment && message.data.post" :to="`/posts/${message.data.post._id}#comments-${message.data.comment._id}`">your comment</TextLink>
          <span v-else>you, but the comment was deleted :(</span>
        </span>
        <Comment v-if="message.data.comment && message.data.post" class="pt-4" :comment="message.data.comment" :standalone="true" />
      </div>
      <div v-else-if="message.type == 'comment_mention'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          mentioned you in
          <TextLink v-if="message.data.comment && message.data.post" :to="`/posts/${message.data.post._id}#comments-${message.data.comment._id}`">their comment</TextLink>
          <span v-else>a deleted comment :(</span>
        </span>
        <Comment v-if="message.data.comment && message.data.post" class="pt-4" :comment="message.data.comment" :standalone="true" />
      </div>
      <div v-else-if="message.type == 'wall_comment'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          commented on
          <TextLink v-if="message.data.comment" :to="`/users/${message.data.wall.name}/wall#comments-${message.data.comment._id}`">your wall</TextLink>
          <span v-else>your wall, but the comment is deleted now :(</span>
        </span>
        <Comment v-if="message.data.comment" class="pt-4" :comment="message.data.comment" :standalone="true" />
      </div>
      <div v-else-if="message.type == 'wall_comment_reply'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          replied to
          <TextLink v-if="message.data.comment" :to="`/users/${message.data.wall.name}/wall#comments-${message.data.comment._id}`">your comment</TextLink>
          <span v-else>you, but the comment was deleted :(</span>
        </span>
        <Comment v-if="message.data.comment" class="pt-4" :comment="message.data.comment" :standalone="true" />
      </div>
      <div v-else-if="message.type == 'wall_comment_mention'">
        <span class="inline-block">
          <TextLink :to="`/users/${message.data.actor.name}`">@{{ message.data.actor.name }}</TextLink>
          mentioned you in
          <TextLink v-if="message.data.comment" :to="`/users/${message.data.wall.name}/wall#comments-${message.data.comment._id}`">their comment</TextLink>
          <span v-else>a deleted comment :(</span>
        </span>
        <Comment v-if="message.data.comment" class="pt-4" :comment="message.data.comment" :standalone="true" />
      </div>
      <div v-else-if="message.type == 'admin_notification'">
        <span class="inline-block prose dark:prose-light" v-html="safeContent">
        </span>
      </div>
      <div v-else>
        <span>unhandled message type "{{ message.type }}"</span>
        <pre>{{ message }}</pre>
      </div>
    </div>
    <div v-if="admin">
      <button @click="deleteMessage"><Icon name="trash" /></button>
    </div>
    <!-- {{ postDate }} -->
  </div>
</template>

<script>
import { sanitize } from "~/lib/sanitizer.js";
import { timeSince } from "~/lib/time-since.js";

export default {
  props: ["message", "admin"],
  methods: {
    select(event) {
      if (event.target.checked) {
        this.$emit("selected");
      } else {
        this.$emit("unselected");
      }
    },
    deleteMessage() {
      // special case for admin view, allow deletion of messages
      this.$emit("delete");
    }
  },
  computed: {
    postDate() {
      return timeSince(this.message.time, {
        cutoff: "4", // days until it should show the full date
      });
    },
    safeContent() {
      // the sanitizer used for admin messages
      return sanitize(this.message.data.content);
    },
  },
};
</script>
