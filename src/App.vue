<script setup>
import { ref, reactive, onMounted } from 'vue'
import { Form, Field, ErrorMessage } from 'vee-validate'
import * as yup from 'yup'

const formSuccess = ref(false)

//Незменяемые справочники не нужно делать реактивными
const citySelectList = [
  { id: 'msc', text: 'Москва' },
  { id: 'spb', text: 'Санкт-Петербург' },
  { id: 'ovb', text: 'Новосибирск' },
  { id: 'svx', text: 'Екатеринбург' },
  { id: 'other', text: 'Другой' },
]

const user = reactive({
  firstname: '',
  lastname: '',
  country: '',
  city: '',
  phone: '',
  email: '',
  password: '',
  showPassword: false,
  confirmPassword: '',
  showConfirmPassword: false,
  comments: '',
  terms: false,
})

const phoneRegExpCommon =
  /^(\+{0,})(\d{0,})([(]{1}\d{1,3}[)]{0,}){0,}(\s?\d+|\+\d{2,3}\s{1}\d+|\d+){1}[\s|-]?\d+([\s|-]?\d+){1,2}(\s){0,}$/gm

const phoneRegExp = /^\+?\d{10,15}$/

const userFormValidationSchema = yup.object({
  firstname: yup.string().required('* Это обязательное поле'),
  lastname: yup.string().required('* Это обязательное поле'),
  country: yup.string().required('* Это обязательное поле'),
  city: yup.string().required('* Это обязательное поле'),
  phone: yup
    .string()
    .required('* Это обязательное поле')
    .matches(phoneRegExp, 'Введите правильный номер телефона'),
  email: yup.string().required('* Это обязательное поле').email('Введите правильный email'),
  password: yup.string().required('* Это обязательное поле').min(6, 'Не менее 6 символов'),
  'confirm-password': yup
    .string()
    .required('* Это обязательное поле')
    .oneOf([yup.ref('password')], 'Введенные пароли не совпадают'),
  comments: yup.string(),
  // terms: (value) => {
  //   if (value && value === 'on') {
  //     return true
  //   }
  //   return 'Необходимо согласиться с условиями'
  // },
  terms: yup.bool().required('Необходимо согласиться с условиями'),
  //.oneOf([true], 'Необходимо согласиться с условиями'),
})

function togglePasswordVisibility() {
  user.showPassword = !user.showPassword
}

//альтернативный способ
function toggleConfirmPasswordVisibility() {
  user.showConfirmPassword = !user.showConfirmPassword

  //Через ref недоступно, элемент Filed это сложный объект
  // const confirmPasswordFileld = useTemplateRef('confirm-password')

  //Через js
  const confirmPasswordFileld = document.getElementById('confirm-password')
  confirmPasswordFileld.type = confirmPasswordFileld.type === 'password' ? 'text' : 'password'
}

function termsHandleChange(event) {
  alert(event.target.checked)
  user.terms = event.target.checked
}

function register(values) {
  formSuccess.value = true
  console.log(JSON.stringify(values, null, 2))
  //alert('Регистрация прошла успешно!')
}
</script>

<template>
  <div class="container">
    <h1 class="title">Регистрация</h1>
    <Form
      v-if="!formSuccess"
      class="registration-form"
      v-bind:validationSchema="userFormValidationSchema"
      @submit="register"
    >
      <div class="form-group">
        <label class="form-label" for="firstname">Имя *</label>
        <Field
          class="form-control"
          name="firstname"
          type="text"
          id="firstname"
          v-model="user.firstname"
        />
        <ErrorMessage name="firstname" class="message message--error" />
      </div>
      <div class="form-group">
        <label class="form-label" for="lastname">Фамилия *</label>
        <Field
          class="form-control"
          name="lastname"
          type="text"
          id="lastname"
          v-model="user.lastname"
        />
        <ErrorMessage name="lastname" class="message message--error" />
      </div>
      <div class="form-group">
        <label class="form-label" for="country">Страна/Регион *</label>
        <Field
          class="form-control"
          name="country"
          type="text"
          id="country"
          v-model="user.country"
        />
        <ErrorMessage name="country" class="message message--error" />
      </div>
      <div class="form-group">
        <label class="form-label" for="city">Город *</label>
        <div class="custom-select">
          <Field as="select" class="form-control" id="city" name="city" v-model="user.city">
            <option value="" disabled selected>Выберите город</option>
            <option v-for="city in citySelectList" :key="`city_${city.id}`" :value="city.id">
              {{ city.text }}
            </option>
            <!-- <option value="spb">Санкт-Петербург</option>
            <option value="ovb">Новосибирск</option>
            <option value="svx">Екатеринбург</option>
            <option value="other">Другой</option> -->
          </Field>
          <ErrorMessage name="city" class="message message--error" />
          <!-- <v-select class="form-control" id="city">
            <option value="" disabled selected>Выберите город</option>
          </v-select> -->
        </div>
      </div>
      <div class="form-group">
        <label class="form-label" for="phone">Телефон *</label>
        <Field class="form-control" type="tel" name="phone" id="phone" v-model="user.phone" />
        <ErrorMessage name="phone" class="message message--error" />
      </div>
      <div class="form-group">
        <label class="form-label" for="email">Email *</label>
        <Field class="form-control" type="email" name="email" id="email" v-model="user.email" />
        <ErrorMessage name="email" class="message message--error" />
      </div>
      <div class="form-group form-group--password">
        <label class="form-label" for="password">Пароль *</label>
        <Field
          id="password"
          name="password"
          class="form-control"
          :type="user.showPassword ? 'text' : 'password'"
          v-model="user.password"
        />
        <button class="btn-icon btn-icon--password" type="button" @click="togglePasswordVisibility">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
            <path
              d="M12 4.5C6.5 4.5 2 9 2 12s4.5 7.5 10 7.5S22 14.5 22 12s-4.5-7.5-10-7.5zm0 13c-3.1 0-5.5-2.4-5.5-5.5S8.9 7.5 12 7.5s5.5 2.4 5.5 5.5S15.1 17.5 12 17.5z"
            />
            <path d="M12 10c-1.1 0-2 .9-2 2s.9 2 2 2c1.1 0 2-.9 2-2s-.9-2-2-2z" />
          </svg>
        </button>
        <ErrorMessage name="password" class="message message--error" />
      </div>
      <div class="form-group form-group--password">
        <label class="form-label" for="confirm-password">Подтвердите пароль *</label>
        <Field
          class="form-control"
          type="password"
          name="confirm-password"
          id="confirm-password"
          v-model="user.confirmPassword"
        />
        <button
          class="btn-icon btn-icon--password"
          type="button"
          @click="toggleConfirmPasswordVisibility"
        >
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
            <path
              d="M12 4.5C6.5 4.5 2 9 2 12s4.5 7.5 10 7.5S22 14.5 22 12s-4.5-7.5-10-7.5zm0 13c-3.1 0-5.5-2.4-5.5-5.5S8.9 7.5 12 7.5s5.5 2.4 5.5 5.5S15.1 17.5 12 17.5z"
            />
            <path d="M12 10c-1.1 0-2 .9-2 2s.9 2 2 2c1.1 0 2-.9 2-2s-.9-2-2-2z" />
          </svg>
        </button>
        <ErrorMessage name="confirm-password" class="message message--error" />
      </div>
      <div class="form-group form-group--full-width">
        <label class="form-label" for="comments">Дополнительная информация</label>
        <Field
          as="textarea"
          class="form-control"
          id="comments"
          name="comments"
          v-model="user.comments"
        />
        <ErrorMessage name="comments" class="message message--error" />
      </div>
      <div class="form-group form-group--full-width">
        <label class="form-label form-label--checkbox" for="terms">
          <!-- <Field
            type="checkbox"
            id="terms"
            name="terms"
            :value="user.terms"
            :checked="user.terms"
            @change="termsHandleChange"
          /> -->
          <Field
            type="checkbox"
            id="terms"
            name="terms"
            v-model="user.terms"
            :value="true"
            :unchecked-value="false"
          />
          Я согласен c условиями пользования и политикой конфиденциальности
        </label>
        <ErrorMessage name="terms" class="message message--error" />
      </div>
      <button class="btn" type="submit">Зарегистрироваться</button>
      <button class="btn" type="reset">Очистить форму</button>
    </Form>
    <div v-else class="message message--success">Регистрация прошла успешно!</div>
  </div>
</template>

<style src="./App.css"></style>
