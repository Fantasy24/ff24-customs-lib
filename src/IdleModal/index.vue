<template>
  <div/>
</template>
<script>
import Vue from 'vue'
import {SESSION_TIMEOUT, WHITE_LIST} from 'ff24-js/src/utils/Constant'
import router from '@/router/routerFactory'
import store from '@/store'

let eventsHub = new Vue({store})
export default {
  created() {
    resetTimer()
  }
}

const ACTIVE = 'ACTIVE'
const DEACTIVE = 'DEACTIVE'

const events = [
  'mousedown',
  'mousemove',
  'keypress',
  'scroll',
  'touchstart'
]

/* Check storeage trigger action*/
window.addEventListener('storage', (e) => {
  e.key === ACTIVE && resetTimer()
  e.key === DEACTIVE && funcLogout()
})

events.forEach(function (name) {
  document.addEventListener(name, resetTimer, true)
  document.addEventListener(name, incr, true)
})

/* Clear cache và hiển thị popup session expired */
function funcLogout() {
  if (checkValidEvent()) {
    events.forEach(function (name) {
      document.removeEventListener(name, resetTimer, true)
      document.removeEventListener(name, incr, true)
    })
    localStorage.setItem(DEACTIVE, true)
    popupWarning()
  }
}

/* Check nếu không ACTIVE trong khoảng time SESSION_TIMEOUT thì logout*/
let to

function resetTimer() {
  clearTimeout(to)
  to = setTimeout(() => {
    funcLogout()
  }, SESSION_TIMEOUT)
}

/* Khi đang active thì ghi time vào localstorage */
function incr() {
  if (checkValidEvent()) {
    const active = localStorage.getItem(ACTIVE) || new Date().getTime()
    localStorage.setItem(ACTIVE, +active + 1)
  }
}

/* Hiển thị popup session expired */
function popupWarning() {
  if (checkValidEvent()) {
    eventsHub.$store.dispatch('user/resetToken')
    eventsHub.$alert('<strong style="color: #f54848">Phiên làm việc đã hết hạn!<br>Tải lại trang (F5) và đăng nhập lại để tiếp tục!</strong>', '', {
      confirmButtonText: 'Tải lại',
      dangerouslyUseHTMLString: true,
      closeOnHashChange: true,
      type: 'warning',
      center: true,
      showClose: false,
      closeOnClickModal: false,
      closeOnPressEscape: false,
      callback: () => {
        location.reload()
      }
    })
  }
}

function checkValidEvent() {
  const currentPath = router.history.current.path
  return WHITE_LIST.indexOf(currentPath) === -1
}


</script>
