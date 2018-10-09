<template>
  <div class="attachments" v-if="attachments.length">
    <div class="attachment" v-for="(attachment, index) in attachments" :key="index">
      <img class="delete-icon" width="16px" src="@/assets/deleteIcon.svg" alt="Delete Icon" @click="emitDeleteEvent(index)">
      <img class="attachment-img" v-if="attachment.base64" :style="{ 'background-image': 'url(' + attachment.base64 +')' }" />
      <div class="preview-name" v-else>
        <img src="@/assets/paperclip.svg" alt="Delete Icon">
        <div> {{ attachment.name }} </div>
      </div>
    </div>
  </div>  
</template>

<script>
export default {
  name: 'AttachmentBox',
  props: [ 'attachments' ],
  data() {
    return {
    }
  },
  methods: {
    emitDeleteEvent(index){
      console.log(index)
      this.$emit('attachmentDelete', index)
    }
  }
}
</script>

<style scoped>

.attachments{
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
}
.attachment{
  position: relative;
  margin: 1px;
  margin-left: 0px;
  min-width: 120px;
  max-height: 120px;
  box-shadow: 0 0 0 1px white, 0 0 0 2px #ebebeb;
}

.attachment-img{
  width: 120px;
  height: 120px;
  background-position: center center;
  background-repeat: no-repeat;
  overflow: hidden;
}

.delete-icon{
  background-color: var(--light-bg); 
  position: absolute;
  right: 0;
}

.preview-name{
  width: 100%;
  height: 40px;
  color: var(--text-grey);
  font-weight: 550;
  font-family: 'Roboto Slab', serif;
  display: flex;
  flex-direction: row;
}
.preview-name > img{
  margin: 0.6rem;
}

.preview-name > div{
  white-space: nowrap;
  margin: 0.75rem;
  margin-right: 24px;
}
</style>


