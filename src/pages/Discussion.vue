<template>
  <div class="discussion">
    <back-to-discussions></back-to-discussions>
    <div v-if="discussion">
      <discussion-post :post="discussion" :is-original-post="true"></discussion-post>
      <discussion-post
        v-for="comment of comments" :post="comment"
        :key="comment.name"
      ></discussion-post>
      <create-discussion-post
        :post="userPost" :inactive="!addCommentActive"
        v-on:post-value="addComment"
      ></create-discussion-post>
    </div>
  </div>
</template>

<script>
import frappe from 'frappejs';
import BackToDiscussions from '@/components/BackToDiscussions';
import DiscussionPost from '@/components/DiscussionPost';
import CreateDiscussionPost from '@/components/CreateDiscussionPost';

export default {
  name: 'Discussion',
  components: {
    BackToDiscussions,
    DiscussionPost,
    CreateDiscussionPost,
  },
  data() {
    return {
      discussion: null,
      comments: [],
      userPost: {
        content: '',
        owner: frappe.session ? frappe.session.fullName : 'Guest',
      },
      addCommentActive: false,
    };
  },
  async beforeCreate() {
    const name = this.$route.params.name;
    const discussion = await frappe.getDoc('DiscussionBoard', name);
    this.comments = await this.getComments(name);

    let attachments = await frappe.db.getAll({
      doctype: 'File',
      fields: ['name', 'filename', 'mimetype', 'size'],
      filters: {
        referenceDoctype: 'DiscussionBoard',
        referenceName: name
      }
    });
    discussion.attachments = attachments;
    this.discussion = discussion;
  },
  methods: {
    async addComment(title, content, attachments) {
      const doc = frappe.newDoc({
        doctype: 'DiscussionBoardMessage',
        creation: new Date().toISOString(),
        owner: frappe.session.user,
        discussionBoard: this.discussion.name,
        content,
        attachments
      });

      await doc.insert();
      this.addNewComment(doc);
      this.addCommentActive = false;
    },
    async addNewComment(comment) {
      let newComment = comment;
      let commentAttachments = await frappe.db.getAll({
        doctype: 'File',
        fields: ['name', 'filename', 'mimetype', 'size'],
        filters: {
          referenceDoctype: 'DiscussionBoardMessage',
          referenceName: comment.name
        }
      });
      newComment.attachments = commentAttachments
      this.comments.push(newComment);
    },
    async getComments(discussionName){
      let comments = await frappe.db.getAll({
        doctype: 'DiscussionBoardMessage',
        fields: ['name', 'content', 'owner', 'creation', 'modified'],
        filters: {
          discussionBoard: discussionName,
        },
        orderBy: 'creation',
        order: 'asc',
      });

      comments.forEach( async (comment) => {
        let commentAttachments = await frappe.db.getAll({
          doctype: 'File',
          fields: ['name', 'filename', 'mimetype', 'size'],
          filters: {
            referenceDoctype: 'DiscussionBoardMessage',
            referenceName: comment.name
          }
        });
        comment.attachments = commentAttachments
      })
      return comments;
    }
  },
};
</script>

<style scoped>
.discussion {
  max-width: 60%;
  margin: 0 auto;
  padding-top: 2rem;
}
</style>
