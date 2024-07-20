<template>
  <div>
    <!-- Основное модальное окно -->
    <div class="modal-container" v-if="isModalVisible">
      <div class="modal-body">
        <h2>Добавить организацию</h2>
        <form @submit.prevent="handleFormSubmit">
          <div class="field-group">
            <label> Название: </label>
            <input
              v-model="orgName"
              @input="checkName"
              required
              placeholder="Введите название организации"
            />
          </div>
          <div class="field-group">
            <label> ФИО директора:</label>
            <div class="input-wrapper">
              <input
                v-model="directorName"
                @input="checkDirector"
                required
                placeholder="Денисов Денис Олегович"
              />
              <span
                v-if="!isDirectorValid"
                class="warning-icon"
                @click="showDirectorWarning = true"
                >&#9888;</span
              >
            </div>
          </div>
          <div class="field-group">
            <label> Номер телефона:</label>
            <div class="input-wrapper">
              <input
                v-model="phoneNumber"
                @input="formatPhoneNumber"
                required
                placeholder="+79957556983"
              />
              <span
                v-if="!isPhoneValid"
                class="warning-icon"
                @click="showPhoneWarning = true"
                >&#9888;</span
              >
            </div>
          </div>
          <div class="button-group">
            <button
              class="submit-btn"
              type="submit"
              :disabled="!isFormComplete"
            >
              ОК
            </button>
            <button type="button" class="cancel-btn" @click="$emit('close')">
              Отмена
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Уведомление об ошибке для ФИО -->
    <div v-if="showDirectorWarning" class="warning-popup">
      <div class="warning-content">
        <span class="close-btn" @click="showDirectorWarning = false"
          >&times;</span
        >
        <p>
          Введите корректное ФИО директора. Убедитесь, что оно состоит из трех
          слов и не содержит латинских символов.
        </p>
      </div>
    </div>

    <!-- Уведомление об ошибке для телефона -->
    <div v-if="showPhoneWarning" class="warning-popup">
      <div class="warning-content">
        <span class="close-btn" @click="showPhoneWarning = false">&times;</span>
        <p>Введите корректный номер телефона в формате +79XXXXXXXXX.</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      orgName: "",
      directorName: "",
      phoneNumber: "",
      isNameValid: true,
      isDirectorValid: true,
      isPhoneValid: true,
      isModalVisible: true, // Показ модального окна
      showDirectorWarning: false,
      showPhoneWarning: false,
    };
  },
  computed: {
    isFormComplete() {
      // Являются ли данные корректны
      return (
        this.orgName &&
        this.directorName &&
        this.phoneNumber &&
        this.isDirectorValid &&
        this.isPhoneValid
      );
    },
  },
  methods: {
    checkName() {
      this.orgName = this.orgName.replace(/[^a-zA-Zа-яА-ЯёЁ0-9\s]/g, ""); // Удаление некорректных символов
      this.isNameValid = this.orgName.trim().length > 0; // Проверка валидности названия
    },
    checkDirector() {
      const hasLatinChars = /[a-zA-Z]/.test(this.directorName); // Проверка на наличие латинских символов
      if (hasLatinChars) {
        // Удаление латинских символов (если они есть)
        this.directorName = this.directorName.replace(/[a-zA-Z]/g, "");
      }
      this.directorName = this.directorName.replace(/[^а-яА-ЯёЁ\s\-']/g, ""); // Удаление некорректных символов
      const words = this.directorName.trim().split(/\s+/); // Проверка валидности имени
      this.isDirectorValid =
        words.length === 3 && words.every((word) => !!word);
    },

    formatPhoneNumber() {
      if (!this.phoneNumber.startsWith("+7")) {
        // Форматирование номера телефона
        this.phoneNumber = "+79" + this.phoneNumber.replace(/[^\d]/g, "");
      } else {
        this.phoneNumber =
          "+" + this.phoneNumber.slice(1).replace(/[^\d]/g, "");
      }
      const phonePattern = /^\+79\d{9}$/; // Проверка валидности номера телефона
      this.isPhoneValid = phonePattern.test(this.phoneNumber);
    },

    handleFormSubmit() {
      if (this.isFormComplete) {
        // Проверка завершенности формы
        this.$emit("add", {
          name: this.orgName,
          director: this.directorName,
          phone: this.phoneNumber,
        });
      } else {
        // Предупреждение
        if (!this.isDirectorValid) this.showDirectorWarning = true;
        if (!this.isPhoneValid) this.showPhoneWarning = true;
      }
    },

    closeModal() {
      this.isModalVisible = false;
    },
  },
};
</script>

<style scoped>
.modal-container {
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

.modal-body {
  background-color: white;
  padding: 20px;
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 400px;
  position: relative;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 30px;
  cursor: pointer;
}

.close-btn:hover {
  color: red;
}

h2 {
  margin-top: 0;
}

form {
  display: flex;
  flex-direction: column;
}

.field-group {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

label {
  width: 120px;
  text-align: center;
}

.input-wrapper {
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

.warning-icon {
  margin-left: 10px;
  font-size: 18px;
  color: red;
  cursor: pointer;
}

.button-group {
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

button.cancel-btn {
  background-color: #dc3545;
  color: white;
}

button.cancel-btn:hover {
  background-color: #c82333;
}

/* Уведомление об ошибке */
.warning-popup {
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

.warning-content {
  background-color: white;
  padding: 20px;
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 300px;
  position: relative;
}

.warning-content .close-btn {
  top: 10px;
  right: 10px;
}

.warning-content p {
  margin: 0;
  color: red;
  font-size: 16px;
}
</style>
