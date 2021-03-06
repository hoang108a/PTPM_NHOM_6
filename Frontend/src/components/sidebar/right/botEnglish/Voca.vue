<template>
  <section class="voca-container">
    <div style="display: flex; justify-content: space-between;">
      <ButtonAdd :badWord="badWord" @addDone="addDone"/>
      <ButtonEdit :badWord="badWord" @editDone="editDone" />
      <div>
        <a-popconfirm
          title="Bạn chắc chắn muốn xóa chứ？"
          okText="Xóa luôn"
          cancelText="Thôi"
          @confirm="confirmDelete"
        >
          <a-button type="danger"><a-icon type="delete" />Xóa</a-button>
        </a-popconfirm>
      </div>
    </div>
    <div class="content">
      <div class="text-voca">
        <span v-if="badWord.badWord !== ''">{{ badWord.badWord }}</span>
        <span v-else>Nơi lưu những từ? </span>
      </div>
      <div class="text-spell">
        <span v-if="badWord.badWord !== ''">{{ badWord.explain }}</span>
        <span v-else>Đã thuộc</span>
      </div>
    </div>
    <div class="bottom">
      <a-button-group style="display: flex;">
        <a-button @click="run(false)" type="primary"> <a-icon type="left" /></a-button>
        <a-button @click="run(true)" type="primary"><a-icon type="right" /></a-button>
      </a-button-group>
      <a-button class="btn" @click="isRandom = !isRandom" :type="isRandom ? 'primary' : 'danger'">Ngẫu nhiên</a-button>
      <a-button class="btn" @click="isAuto = !isAuto" :type="isAuto ? 'primary' : 'danger'">Tự động</a-button>
    </div>
  </section>
</template>

<script>

import { API } from '@/constants/api'
import { jsonHeader } from '@/utils/fetchTool'
import ButtonAdd from './ButtonAdd'
import ButtonEdit from './ButtonEdit'
import { mapGetters, mapActions } from 'vuex'

export default {
  name: 'voca',
  data () {
    return {
      isRandom: false,
      isAuto: false,
      badWord: {
        badWord: ''
      },
      user: null,
      index: 0,
      interval: null,
      isNext: true
    }
  },
  components: {
    ButtonAdd,
    ButtonEdit
  },
  props: ['socket'],
  computed: {
    ...mapGetters(['badWords'])
  },
  methods: {
    ...mapActions(['getBadWords']),
    confirmDelete () {
      fetch(`${API.DELETE_BADWORD}/${this.badWord._id}`, {
        headers: jsonHeader.headers,
        method: 'delete'
      }).then((response) => response.json())
        .then((res) => {
          if (res.code === 200) {
            this.$message.success(res.data.message)
            this.deleteDone()
          } else {
            this.$message.error(res.data.message)
          }
        })
        .catch((err) => {
          console.log(err)
        })
    },
    run (isNext) {
      console.log(this.badWords)
      if (this.isRandom) {
        this.index = Math.floor(Math.random() * this.badWords.length)
      } else {
        if (isNext) {
          if (this.index < this.badWords.length - 1) {
            this.index++
          } else {
            this.isNext = false
          }
        } else {
          if (this.index > 0) {
            this.index--
          } else {
            this.isNext = true
          }
        }
      }
      if (this.badWords.length > 0) {
        this.badWord = this.badWords[this.index]
      } else {
        this.badWord = {
          badWord: 'Empty'
        }
      }
    },
    editDone () {
      this.getBadWords({
        email: this.user.email
      })
      this.run()
    },
    addDone () {
      this.getBadWords({
        email: this.user.email
      })
      this.run()
    },
    deleteDone () {
      this.getBadWords({
        email: this.user.email
      })
      this.run()
    },
    connectSocket () {
      this.socket.on('serverUpdateBadWord', () => {
        this.getBadWords({
          email: this.user.email
        })
        this.run()
      })
    }
  },
  beforeMount () {
    this.user = JSON.parse(localStorage.getItem('user'))
    if (this.user) {
      this.getBadWords({
        email: this.user.email
      })
      this.connectSocket()
    }
  },
  watch: {
    isAuto: function () {
      if (this.isAuto) {
        this.interval = setInterval(() => {
          this.run(this.isNext)
        }, 500)
      } else {
        clearInterval(this.interval)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .voca-container {
    position: relative;
    .content {
      position: absolute;
      top: 50%;
      right: 50%;
      width: 90%;
      margin: 0 auto;
      text-align: center;
      transform: translate(50%, -50%);
      .text-voca {
        span {
          font-size: 40px;
          color: white;
        }
      }
      .text-spell {
        span {
          font-size: 20px;
          color: white;
        }
      }
    }
    .bottom {
      display: flex;
      position: absolute;
      bottom: 0px;
      .btn {
        max-width: 80px;
        margin-left: 5px;
      }
    }
  }
</style>
