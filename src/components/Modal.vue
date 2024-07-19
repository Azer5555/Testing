<template>
  <div>
    <!-- Основное модальное окно -->
    <div class="modal" v-if="showModal">
      <div class="modal-content">
        <h2>Добавить организацию</h2>
        <form @submit.prevent="submitForm">
          <div class="form-group">
            <label> Название: </label>
            <input
              v-model="name"
              @input="validateName"
              required
              placeholder="Введите название организации"
            />
          </div>
          <div class="form-group">
            <label> ФИО директора:</label>
            <div class="input-container">
              <input
                v-model="director"
                @input="validateDirector"
                required
                placeholder="Иванов Иван Иванович"
              />
              <span
                v-if="!isDirectorValid"
                class="error-icon"
                @click="showDirectorError = true"
                >&#9888;</span
              >
            </div>
          </div>
          <div class="form-group">
            <label> Номер телефона:</label>
            <div class="input-container">
              <input
                v-model="phone"
                @input="formatPhone"
                required
                placeholder="+79957556983"
              />
              <span
                v-if="!isPhoneValid"
                class="error-icon"
                @click="showPhoneError = true"
                >&#9888;</span
              >
            </div>
          </div>
          <div class="btn">
            <button class="good" type="submit" :disabled="!isFormValid">
              ОК
            </button>
            <button type="button" class="cancel" @click="$emit('close')">
              Отмена
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Уведомление об ошибке для ФИО -->
    <div v-if="showDirectorError" class="error-modal">
      <div class="error-modal-content">
        <span class="close" @click="showDirectorError = false">&times;</span>
        <p>
          Введите корректное ФИО директора. Убедитесь, что оно состоит из трех
          слов и не содержит латинских символов.
        </p>
      </div>
    </div>

    <!-- Уведомление об ошибке для телефона -->
    <div v-if="showPhoneError" class="error-modal">
      <div class="error-modal-content">
        <span class="close" @click="showPhoneError = false">&times;</span>
        <p>Введите корректный номер телефона в формате +79XXXXXXXXX.</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      director: "",
      phone: "",
      isNameValid: true,
      isDirectorValid: true,
      isPhoneValid: true,
      showModal: true, // Показ модального окна
      showDirectorError: false,
      showPhoneError: false,
    };
  },
  computed: {
    isFormValid() {
      return (
        this.name &&
        this.director &&
        this.phone &&
        this.isDirectorValid &&
        this.isPhoneValid
      );
    },
  },
  methods: {
    validateName() {
      this.name = this.name.replace(/[^a-zA-Zа-яА-ЯёЁ0-9\s]/g, "");
      this.isNameValid = this.name.trim().length > 0;
    },
    validateDirector() {
      const hasLatinChars = /[a-zA-Z]/.test(this.director);
      if (hasLatinChars) {
        this.director = this.director.replace(/[a-zA-Z]/g, "");
      }
      this.director = this.director.replace(/[^а-яА-ЯёЁ\s\-']/g, "");
      const words = this.director.trim().split(/\s+/);
      this.isDirectorValid =
        words.length === 3 && words.every((word) => !!word);
    },

    formatPhone() {
      if (!this.phone.startsWith("+7")) {
        this.phone = "+79" + this.phone.replace(/[^\d]/g, "");
      } else {
        this.phone = "+" + this.phone.slice(1).replace(/[^\d]/g, "");
      }
      const phonePattern = /^\+79\d{9}$/;
      this.isPhoneValid = phonePattern.test(this.phone);
    },

    submitForm() {
      if (this.isFormValid) {
        this.$emit("add", {
          name: this.name,
          director: this.director,
          phone: this.phone,
        });
      } else {
        if (!this.isDirectorValid) this.showDirectorError = true;
        if (!this.isPhoneValid) this.showPhoneError = true;
      }
    },

    closeModal() {
      this.showModal = false;
    },
  },
};
</script>

<style scoped>
.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 400px;
  position: relative;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 30px;
  cursor: pointer;
}

.close:hover {
  color: red;
}

h2 {
  margin-top: 0;
}

form {
  display: flex;
  flex-direction: column;
}

.form-group {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

label {
  width: 120px;
  text-align: center;
}

.input-container {
  display: flex;
  align-items: center;
  position: relative;
}

input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.error-icon {
  margin-left: 10px;
  font-size: 18px;
  color: red;
  cursor: pointer;
}

.btn {
  display: flex;
  justify-content: center;
  gap: 15px;
}

button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

button.cancel {
  background-color: #dc3545;
  color: white;
}

button.cancel:hover {
  background-color: #c82333;
}

/* Уведомление об ошибке */
.error-modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.error-modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 300px;
  position: relative;
}

.error-modal-content .close {
  top: 10px;
  right: 10px;
}

.error-modal-content p {
  margin: 0;
  color: red;
  font-size: 16px;
}
</style>
