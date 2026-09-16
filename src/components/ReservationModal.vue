<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue'

const props = defineProps({
  open: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['close'])

const form = reactive({
  name: '',
  phone: '',
  date: '',
  time: '',
  guests: '',
  occasion: '',
  notes: ''
})

const errors = reactive({
  name: '',
  phone: '',
  date: '',
  time: '',
  guests: ''
})

const submitted = ref(false)
const panel = ref(null)
const nameInput = ref(null)
const successHeading = ref(null)
let lastFocusedElement = null

const timeSlots = ['11:30', '12:00', '12:30', '17:30', '18:00', '18:30', '19:00', '19:30']
const occasions = ['家庭聚餐', '商务宴请', '朋友小聚', '生日寿宴', '其他']

const today = computed(() => getLocalDateString())

function getLocalDateString() {
  const now = new Date()
  const year = now.getFullYear()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const day = String(now.getDate()).padStart(2, '0')
  return year + '-' + month + '-' + day
}

function resetForm() {
  form.name = ''
  form.phone = ''
  form.date = getLocalDateString()
  form.time = ''
  form.guests = ''
  form.occasion = ''
  form.notes = ''
  clearAllErrors()
}

function clearError(field) {
  errors[field] = ''
}

function clearAllErrors() {
  Object.keys(errors).forEach((field) => {
    errors[field] = ''
  })
}

function handlePhoneInput() {
  form.phone = form.phone.replace(/\D/g, '').slice(0, 11)
  clearError('phone')
}

function handleGuestsChange() {
  clearError('guests')
  if (form.guests === 13) {
    errors.guests = '超过 12 人请致电 021-6288 8899 安排宴席。'
  }
}

function validateForm() {
  clearAllErrors()
  let firstInvalidField = ''

  if (!form.name.trim()) {
    errors.name = '请输入您的姓名或称呼。'
    firstInvalidField = 'name'
  }

  if (!/^1[3-9]\d{9}$/.test(form.phone.trim())) {
    errors.phone = '请输入有效的 11 位中国大陆手机号。'
    if (!firstInvalidField) firstInvalidField = 'phone'
  }

  if (!form.date) {
    errors.date = '请选择到店日期。'
    if (!firstInvalidField) firstInvalidField = 'date'
  } else if (form.date < today.value) {
    errors.date = '到店日期不能早于今天。'
    if (!firstInvalidField) firstInvalidField = 'date'
  }

  if (!form.time) {
    errors.time = '请选择到店时间。'
    if (!firstInvalidField) firstInvalidField = 'time'
  }

  if (!form.guests) {
    errors.guests = '请选择用餐人数。'
    if (!firstInvalidField) firstInvalidField = 'guests'
  } else if (form.guests === 13) {
    errors.guests = '超过 12 人请致电 021-6288 8899 安排宴席。'
    if (!firstInvalidField) firstInvalidField = 'guests'
  }

  return firstInvalidField
}

async function handleSubmit() {
  const firstInvalidField = validateForm()
  if (firstInvalidField) {
    await nextTick()
    document.getElementById('reservation-' + firstInvalidField)?.focus()
    return
  }

  resetForm()
  submitted.value = true
  await nextTick()
  successHeading.value?.focus()
}

function closeModal() {
  emit('close')
}

function getFocusableElements() {
  if (!panel.value) return []
  const selector = 'button:not([disabled]), [href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'
  return Array.from(panel.value.querySelectorAll(selector)).filter((element) => element.offsetParent !== null)
}

function handleKeydown(event) {
  if (!props.open) return

  if (event.key === 'Escape') {
    event.preventDefault()
    closeModal()
    return
  }

  if (event.key !== 'Tab') return

  const focusableElements = getFocusableElements()
  if (!focusableElements.length) {
    event.preventDefault()
    return
  }

  const firstElement = focusableElements[0]
  const lastElement = focusableElements[focusableElements.length - 1]

  if (event.shiftKey && document.activeElement === firstElement) {
    event.preventDefault()
    lastElement.focus()
  } else if (!event.shiftKey && document.activeElement === lastElement) {
    event.preventDefault()
    firstElement.focus()
  }
}

watch(() => props.open, async (isOpen) => {
  if (isOpen) {
    lastFocusedElement = document.activeElement
    submitted.value = false
    resetForm()
    document.body.classList.add('modal-open')
    await nextTick()
    nameInput.value?.focus()
    return
  }

  document.body.classList.remove('modal-open')
  await nextTick()
  if (lastFocusedElement && typeof lastFocusedElement.focus === 'function') {
    lastFocusedElement.focus()
  }
})

onMounted(() => {
  document.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleKeydown)
  document.body.classList.remove('modal-open')
})
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div
        v-if="open"
        id="reservation-modal"
        class="modal is-open"
        role="dialog"
        aria-modal="true"
        aria-labelledby="reservation-title"
      >
        <div class="modal-backdrop" @click="closeModal"></div>
        <div ref="panel" class="modal-panel">
          <button class="modal-close" type="button" aria-label="关闭订座窗口" @click="closeModal">×</button>

          <div v-if="!submitted" id="reservation-form-view">
            <div class="modal-heading">
              <p class="eyebrow">Table Reservation</p>
              <h2 id="reservation-title">预订一席</h2>
              <p>请填写以下信息，我们将根据用餐需求为您安排合适的席位。</p>
            </div>

            <form class="reservation-form" novalidate @submit.prevent="handleSubmit">
              <div class="form-grid">
                <label class="form-field" for="reservation-name" :class="{ 'has-error': errors.name }">
                  <span class="form-label">姓名 <b>*</b></span>
                  <input
                    id="reservation-name"
                    ref="nameInput"
                    v-model="form.name"
                    name="name"
                    type="text"
                    autocomplete="name"
                    placeholder="请输入您的称呼"
                    :aria-invalid="Boolean(errors.name)"
                    aria-describedby="reservation-name-error"
                    @input="clearError('name')"
                  >
                  <span id="reservation-name-error" class="field-error" aria-live="polite">{{ errors.name }}</span>
                </label>

                <label class="form-field" for="reservation-phone" :class="{ 'has-error': errors.phone }">
                  <span class="form-label">手机号 <b>*</b></span>
                  <input
                    id="reservation-phone"
                    v-model="form.phone"
                    name="phone"
                    type="tel"
                    inputmode="numeric"
                    autocomplete="tel"
                    maxlength="11"
                    placeholder="请输入 11 位手机号"
                    :aria-invalid="Boolean(errors.phone)"
                    aria-describedby="reservation-phone-error"
                    @input="handlePhoneInput"
                  >
                  <span id="reservation-phone-error" class="field-error" aria-live="polite">{{ errors.phone }}</span>
                </label>

                <label class="form-field" for="reservation-date" :class="{ 'has-error': errors.date }">
                  <span class="form-label">到店日期 <b>*</b></span>
                  <input
                    id="reservation-date"
                    v-model="form.date"
                    name="date"
                    type="date"
                    :min="today"
                    :aria-invalid="Boolean(errors.date)"
                    aria-describedby="reservation-date-error"
                    @input="clearError('date')"
                  >
                  <span id="reservation-date-error" class="field-error" aria-live="polite">{{ errors.date }}</span>
                </label>

                <label class="form-field" for="reservation-time" :class="{ 'has-error': errors.time }">
                  <span class="form-label">到店时间 <b>*</b></span>
                  <select
                    id="reservation-time"
                    v-model="form.time"
                    name="time"
                    :aria-invalid="Boolean(errors.time)"
                    aria-describedby="reservation-time-error"
                    @change="clearError('time')"
                  >
                    <option value="">请选择时间</option>
                    <option v-for="time in timeSlots" :key="time" :value="time">{{ time }}</option>
                  </select>
                  <span id="reservation-time-error" class="field-error" aria-live="polite">{{ errors.time }}</span>
                </label>

                <label class="form-field" for="reservation-guests" :class="{ 'has-error': errors.guests }">
                  <span class="form-label">用餐人数 <b>*</b></span>
                  <select
                    id="reservation-guests"
                    v-model="form.guests"
                    name="guests"
                    :aria-invalid="Boolean(errors.guests)"
                    aria-describedby="reservation-guests-error"
                    @change="handleGuestsChange"
                  >
                    <option value="">请选择人数</option>
                    <option v-for="count in 12" :key="count" :value="count">{{ count }} 位</option>
                    <option :value="13">12 位以上（请致电）</option>
                  </select>
                  <span id="reservation-guests-error" class="field-error" aria-live="polite">{{ errors.guests }}</span>
                </label>

                <label class="form-field" for="reservation-occasion">
                  <span class="form-label">用餐场景</span>
                  <select id="reservation-occasion" v-model="form.occasion" name="occasion">
                    <option value="">请选择场景（选填）</option>
                    <option v-for="occasion in occasions" :key="occasion" :value="occasion">{{ occasion }}</option>
                  </select>
                  <span class="field-error" aria-hidden="true"></span>
                </label>

                <label class="form-field form-field--full" for="reservation-notes">
                  <span class="form-label">特殊需求</span>
                  <textarea
                    id="reservation-notes"
                    v-model="form.notes"
                    name="notes"
                    maxlength="200"
                    placeholder="如忌口、儿童餐椅、指定雅间等，可在此说明"
                  ></textarea>
                  <span class="field-error" aria-hidden="true"></span>
                </label>

                <p class="form-note">提交即表示您同意我们仅将以上信息用于本次预订联系。此页面为前端演示，不会向服务器发送数据。</p>

                <button class="btn btn--primary form-submit" type="submit">
                  提交预订需求 <span class="arrow" aria-hidden="true">→</span>
                </button>
              </div>
            </form>
          </div>

          <div v-else ref="successHeading" class="reservation-success" tabindex="-1">
            <div class="success-mark" aria-hidden="true">✓</div>
            <h2>预订需求已记录</h2>
            <p>当前为前端演示模式。为确保席位确认，请致电门店，我们将为您完成最终预订。</p>
            <a class="success-phone" href="tel:02162888899">021-6288 8899</a>
            <div>
              <button class="btn btn--primary success-close" type="button" @click="closeModal">关闭窗口</button>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>
