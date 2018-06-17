<template>
  <i-layout>
    <Modal v-model="modal6" :title="modalTitle" :loading="modalLoading" @on-ok="asyncOK">
      <p>{{modalContent}}</p>
    </Modal>
  </i-layout>
</template>
<script>
import navContent from "../../components/navcontent.vue";
import iLayout from "../../components/layout.vue";

export default {
  name: "userManage",
  components: {
    iLayout,
    navContent
  }
};
</script>
