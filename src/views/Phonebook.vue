<template>
  <div id="phonebook">
    <header>
      <span class="title">{{currentFolderName}}</span>
      <div class="right-btn">
        <i class="iconfont icon-pen" @click="showNewContactModal"></i>
      </div>
    </header>
    <div class="container">
      <index></index>
      <div v-for="(item,index) in items" class="wrap">
        <div v-if="!items[index-1] || item.initial !== items[index-1].initial" class="initial" :id="item.initial">{{item.initial}}</div>
        <div class="item" :data-id="item._id" v-finger:long-tap="showDeleteModal">
          <div class="name">{{item.contact}}</div>
          <div class="number">{{item.number}}</div>
        </div>
      </div>
    </div>
    <new-contact-modal ref="NewContactModal"></new-contact-modal>
    <delete-modal ref="DeleteModal"></delete-modal>
  </div>
</template>

<script>
import axios from 'axios'
import api from '../api/api-config.js'
import { mapGetters } from 'vuex'
import Index from '../components/contact/Index.vue'
import NewContactModal from '../components/contact/NewContactModal.vue'
import DeleteModal from '../components/DeleteModal.vue'

import Vue from 'vue'
import AlloyFinger from 'alloyfinger/alloy_finger.js'
import AlloyFingerVue from 'alloyfinger/vue/alloy_finger.vue.js'
Vue.use(AlloyFingerVue, {
  AlloyFinger
})

export default {
  data() {
    return {
      contact: '',
      initial: '',
      number: '',
      items: [],
      selectedItem: ''
    }
  },
  activated() {
    this.getFolderContents()
  },
  computed: mapGetters({
    currentFolder: 'getCurrentFolder',
    currentFolderName: 'getCurrentFolderName'
  }),
  components: {
    Index,
    DeleteModal,
    NewContactModal
  },
  methods: {
    showNewContactModal() {
      this.$refs.NewContactModal.isModalShow = true;
    },
    showDeleteModal(e) {
      this.$refs.DeleteModal.isModalShow = true;
      if (e.target.dataset.id) {
        this.selectedItem = e.target.dataset.id;
      } else {
        this.selectedItem = e.target.parentNode.dataset.id;
      }
      console.log(this.selectedItem)
    },
    getFolderContents() {
      console.log(this.currentFolder)
      axios.get(api.getContactContents + this.currentFolder)
        .then(res => {
          if (res.data.code === 0) {
            console.log(res.data.data)
            this.items = res.data.data
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    deleteItem() {
      axios.delete(api.deleteContact + this.selectedItem)
        .then(res => {
          if (res.data.code === 0) {
            console.log(res)
            this.getFolderContents()
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    }
  }
}
</script>

<style lang="less" scoped>
@import '../less/common.less';
header {
  .commonheader(@bgcolor: #fff, @fontcolor: @maincolor);
}

.container {
  display: flex;
  justify-content: baseline;
  align-items: center;
  flex-direction: column;
  height: @contactcontainerheight;
  background-color: @maincolor;
  box-sizing: border-box;
  overflow: scroll;
  .wrap {
    width: 70%;
    .item {
      padding: 10px;
      background-color: #fff;
      color: @maincolor;
      border-radius: 5px;
      margin: 5px 0;
      .name {
        font-size: 1.8rem;
        text-align: left;
      }
      .number{
        font-size: .9;
        text-align: right;
      }
    }
    .initial {
      text-align: center;
      font-size: 2rem;
      color: #fff;
      font-weight: bold;
    }
  }
}
</style>