<template>
  <Chat></Chat>
</template>

<script>
import ChatRAGE from './components/ChatRAGE.vue';
export default {
  name: 'App',
  components: {
    Chat: ChatRAGE
  }
}
</script>