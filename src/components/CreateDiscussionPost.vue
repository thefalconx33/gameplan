<template>
  <discussion-post-wrapper>
    <div :class="{post: true, 'is-collapsed': !isActive}">
      <user-avatar :user="post.owner"></user-avatar>
      <div class="body" @drop="onDrop">
        <input class="title" v-if="isOriginalPost"
          ref="title" type="text" placeholder="Title" v-model="title">
        <div class="title" v-else>
          {{ post.title }}
        </div>
        <div class="owner">
          {{ post.owner }}
        </div>
        <content-editor
          v-model="content"
          v-if="isActive"
          @submit="postValue"
          @escape="deactivate"
        />
        <AttachmentBox v-if="attachments.length" :attachments="attachments" @attachmentDelete="handleDelete"/>
        <div class="action" v-if="isActive">
          <span>Cmd + Enter to post</span>
          <div class="message"> {{ currMessage }} </div>
        </div>
        <div class="action" v-if="!isActive" style="margin-top: 0.5rem;">
          <span @click="activate">Add to this discussion</span>
        </div>
      </div>
    </div>
  </discussion-post-wrapper>
</template>

<script>
import DiscussionPostWrapper from '@/components/DiscussionPostWrapper';
import UserAvatar from '@/components/UserAvatar';
import ContentEditor from './ContentEditor';
import AttachmentBox from './AttachmentBox';

export default {
  name: 'CreateDiscussionPost',
  components: {
    DiscussionPostWrapper,
    UserAvatar,
    ContentEditor,
    AttachmentBox
  },
  props: ['post', 'is-original-post', 'inactive'],
  data() {
    return {
      title: '',
      content: '',
      attachments: [],
      currMessage: 'Drag and Drop files to upload',
      active: this.inactive === undefined || this.inactive === false,
    };
  },
  computed: {
    isActive() {
      return this.isOriginalPost || this.active;
    },
  },
  methods: {
    onDrop(event) {
      event.preventDefault();
      this.currMessage = 'Drag and drop files to upload'
      Array.from(event.dataTransfer.files).forEach(file => {
        if(!this.attachments.some( f => f.name === file.name )){ // Check if already attached
          if(file.type.match('image.*')){ //Generate preview using base64 string
            let reader = new FileReader();
            reader.readAsDataURL(file);
            reader.onload = () => {
              file.base64 = reader.result
              this.attachments.push(file)
            };
            reader.onerror = (error) => {
              console.log('Error converting to base64: ', error);
            };
          } else {
            this.attachments.push(file)
          }
        }
      })
    },
    postValue() {
      if(this.title || this.content){
        this.$emit('post-value', this.title, this.content, this.attachments);
        this.content = '';
        this.title = '';
        this.attachments = [];
        this.deactivate();
      }
    },
    activate() {
      this.active = true;
    },
    deactivate() {
      this.active = false;
    },
    handleDelete(index) {
      this.attachments.splice(index, 1);
    }
  },
  mounted() {
    if (this.$refs.title) {
      this.$refs.title.focus();
    }
    document.addEventListener('dragover', (e) => {
      e.preventDefault()
      this.currMessage = 'Drop here to upload'
    })
    document.addEventListener('dragleave', (e) => {
      this.currMessage = 'Drag and drop files to upload'
      e.preventDefault()
    })
    document.addEventListener('drop', (e) => {
      this.currMessage = 'Drag and drop files to upload'
      e.preventDefault()
    })
  },
  beforeDestroy() {
    document.removeEventListener('drop', () => {});
    document.removeEventListener('dragover', () => {});
    document.removeEventListener('dragleave', () => {});
  }
};
</script>

<style scoped>
.post:not(.is-collapsed) {
  background-color: #ffffff;
}

input.title {
  padding: 0;
  border-bottom-color: transparent;
}

.body{
  padding: 0px 8px;
}

.action{
  position: relative;
}

.feedback-btn > span{
  margin-left: 1.6rem;
  font-size: 0.9rem;
  overflow: hidden;
  white-space: nowrap;
  opacity: 0;
  transition: opacity 0.5s cubic-bezier(.34,.6,.12,.99) 0.1s;
}

textarea.content {
  border: none;
  outline: none;
  padding: 0;
  background-color: transparent;
  width: 100%;
  min-height: 5rem;
  overflow: visible;
  resize: none;
  caret-color: var(--text-blue);
}

textarea.content::placeholder {
  color: var(--text-grey);
}

.message{
  color: var(--text-grey);
  font-size: 1.2rem;
  font-weight: 300;
  cursor: pointer;
}
</style>
