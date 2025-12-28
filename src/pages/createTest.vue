<template>
  <div class="test-wrapper">
    <!-- HEADER -->
    <header class="test-header">
      <button @click="$router.back()" class="back-btn">⬅ Назад</button>
      <h1 class="test-title">Створити тест</h1>
      <h3>Кількість питань: {{ qCount }}</h3>
    </header>

    <!-- Назва тесту -->
    <input v-model="title" placeholder="Назва тесту" class="title-input" />

    <div v-for="(q, qIndex) in exercises" :key="qIndex" class="question-block">
      <h4>Питання {{ qIndex + 1 }}</h4>
      <input v-model="q.question" placeholder="Текст питання" class="question-input" />

      <button @click="removeQuestion(qIndex)" class="btn-small btn-remove-question">Видалити питання</button>

      <select v-model="q.type" class="type-select">
        <option value="one">Одне правильне</option>
        <option value="many">Багато правильних</option>
        <option value="pair">Пари</option>
        <option value="enter">Введення</option>
      </select>

      <!-- ANSWERS one/many -->
      <div v-if="q.type === 'one' || q.type === 'many'" class="answers-block">
        <div v-for="(a, aIndex) in q.answers" :key="aIndex" class="answer-row">
          <input v-if="!a.isImage" v-model="a.text" placeholder="Введіть текст" class="answer-input" />

          <div v-if="a.isImage">
            <label v-if="a.preview">
              <img :src="a.preview" class="preview-img" title="Натисніть, щоб змінити" />
              <input class="upload-label-input" type="file" accept="image/*"
                @change="onAnswerImageChange($event, qIndex, aIndex)" :ref="'fileInput-' + qIndex + '-' + aIndex">
            </label>
            <label v-else class="upload-label">
              <span>Select image</span>
              <input class="upload-label-input" type="file" accept="image/*"
                @change="onAnswerImageChange($event, qIndex, aIndex)" :ref="'fileInput-' + qIndex + '-' + aIndex">
            </label>
          </div>

          <label>
            <template v-if="q.type === 'many'">
              <input type="checkbox" v-model="a.correct" /> Правильна
            </template>
            <template v-else>
              <input type="radio" :name="'one-' + qIndex" :value="aIndex" v-model="q.correctAnswerIndex" /> Правильна
            </template>
          </label>

          <label>
            <input type="checkbox" v-model="a.isImage" /> Картинка
          </label>

          <button @click="removeAnswer(qIndex, aIndex)" class="btn-small">Видалити</button>
        </div>
        <button @click="addAnswer(qIndex)" class="btn-small btn-add">Додати варіант</button>
      </div>

      <!-- ENTER -->
      <div v-if="q.type === 'enter'" class="enter-block">
        <div v-for="(r, rIndex) in q.correctAnswers" :key="rIndex" class="enter-row">
          <input v-model="q.correctAnswers[rIndex]" placeholder="Правильна відповідь" class="enter-input" />
          <button @click="removeCorrectAnswer(qIndex, rIndex)" class="btn-small">Видалити</button>
        </div>
        <button @click="addCorrectAnswer(qIndex)" class="btn-small btn-add">Додати відповідь</button>
      </div>

      <!-- PAIRS -->
      <div v-if="q.type === 'pair'" class="pairs-block">
        <div class="pair-columns">
          <!-- Ліва колонка -->
          <div>
            <h6>Ліва колонка</h6>
            <div v-for="(left, lIndex) in q.pairs.left" :key="'left-' + qIndex + '-' + lIndex" class="pair-row">
              <input v-if="!left.isImage" v-model="left.text" placeholder="Лівий елемент" class="pair-input" />

              <div v-if="left.isImage">
                <label v-if="left.preview">
                  <img :src="left.preview" class="preview-img" title="Натисніть, щоб змінити" />
                  <input class="upload-label-input" type="file" accept="image/*"
                    @change="onPairLeftImageChange($event, qIndex, lIndex)" :ref="'leftFile-' + qIndex + '-' + lIndex">
                </label>
                <label v-else class="upload-label">
                  <span>Select image</span>
                  <input class="upload-label-input" type="file" accept="image/*"
                    @change="onPairLeftImageChange($event, qIndex, lIndex)" :ref="'leftFile-' + qIndex + '-' + lIndex">
                </label>
              </div>

              <label>
                <input type="checkbox" v-model="left.isImage" /> Картинка
              </label>

              <select v-model="q.pairs.correctMap[lIndex]">
                <option v-for="(right, rIndex) in q.pairs.right" :key="rIndex" :value="rIndex">
                  {{ rIndex + 1 }}
                </option>
              </select>
              <button @click="removeLeft(qIndex, lIndex)" class="btn-small">Видалити лівий</button>
            </div>
            <button @click="addLeft(qIndex)" class="btn-small btn-add">Додати лівий</button>
          </div>

          <!-- Права колонка -->
          <div>
            <h6>Права колонка</h6>
            <div v-for="(right, rIndex) in q.pairs.right" :key="'right-' + qIndex + '-' + rIndex" class="pair-row">
              <p>{{ rIndex + 1 }}</p>
              <input v-if="!right.isImage" v-model="right.text" placeholder="Правий елемент" class="pair-input" />

              <div v-if="right.isImage">
                <label v-if="right.preview">
                  <img :src="right.preview" class="preview-img" title="Натисніть, щоб змінити" />
                  <input class="upload-label-input" type="file" accept="image/*"
                    @change="onPairRightImageChange($event, qIndex, rIndex)"
                    :ref="'rightFile-' + qIndex + '-' + rIndex">
                </label>
                <label v-else class="upload-label">
                  <span>Select image</span>
                  <input class="upload-label-input" type="file" accept="image/*"
                    @change="onPairRightImageChange($event, qIndex, rIndex)"
                    :ref="'rightFile-' + qIndex + '-' + rIndex">
                </label>
              </div>

              <label>
                <input type="checkbox" v-model="right.isImage" /> Картинка
              </label>

              <button @click="removeRight(qIndex, rIndex)" class="btn-small">Видалити правий</button>
            </div>
            <button @click="addRight(qIndex)" class="btn-small btn-add">Додати зайвий правий</button>
          </div>
        </div>
      </div>
    </div>
    <div class="btn-down">
      <button @click="addQuestion" class="btn-add-question">Додати питання</button>
      <button @click="createTest" class="btn-create-test">Створити тест</button>
    </div>
    <p class="message">{{ message }}</p>
  </div>

  <!-- Модальне вікно створеного тесту -->
  <div v-if="showModal" class="modal-overlay" @click="showModal = false">
    <div class="modal-content animated-modal" @click.stop>
      <h2>🎉 Тест створено!</h2>
      <div class="modal-el">
        <p><strong>Код тесту:</strong> <span class="code-text">{{ testCode }}</span>
          <button @click="copyToClipboard(testCode)" class="btn-copy">📋</button>
        </p>
      </div>
      <div class="modal-el">
        <strong>Посилання на тест: </strong>
        <p>
          <a :href="testLink" target="_blank">{{ testLink }}</a>
          <button @click="copyToClipboard(testLink)" class="btn-copy">📋</button>
        </p>
      </div>
      <button @click="showModal = false" class="btn-close">✖</button>
    </div>
  </div>

</template>

<script>
import axios from "axios";
const BACK_URL = import.meta.env.VITE_BACK_URL;

export default {
  data() {
    return {
      title: "",
      exercises: [
        {
          type: "one",
          question: "",
          answers: [],
          pairs: { left: [], right: [], correctMap: {} },
          correctAnswers: [],
          correctAnswerIndex: null
        }
      ],
      message: "",
      showModal: false,
      testCode: "",
      testLink: "",
    };
  },
  computed: {
    qCount() {
      return this.exercises.length;
    }
  },
  methods: {
    copyToClipboard(text) {
      navigator.clipboard.writeText(text).then(() => {
        this.$root.showToast('Скопійовано');

      }).catch(err => console.error(err));
    },
    addQuestion() {
      this.exercises.push({
        type: "one",
        question: "",
        answers: [],
        pairs: { left: [], right: [], correctMap: {} },
        correctAnswers: [],
        correctAnswerIndex: null
      });
    },

    removeQuestion(qIndex) {
      this.exercises.splice(qIndex, 1);
    },

    addAnswer(qIndex) {
      this.exercises[qIndex].answers.push({ text: "", correct: false, isImage: false, file: null, preview: null });
    },
    removeAnswer(qIndex, aIndex) { this.exercises[qIndex].answers.splice(aIndex, 1); },
    onAnswerImageChange(event, qIndex, aIndex) {
      const file = event.target.files[0];
      const answer = this.exercises[qIndex].answers[aIndex];
      answer.file = file || null;
      answer.preview = file ? URL.createObjectURL(file) : null;
    },
    addCorrectAnswer(qIndex) { this.exercises[qIndex].correctAnswers.push(""); },
    removeCorrectAnswer(qIndex, rIndex) { this.exercises[qIndex].correctAnswers.splice(rIndex, 1); },
    addLeft(qIndex) {
      const q = this.exercises[qIndex];
      q.pairs.left.push({ text: "", isImage: false, file: null });
      q.pairs.correctMap[q.pairs.left.length - 1] = 0;
      if (q.pairs.right.length < q.pairs.left.length) q.pairs.right.push({ text: "", isImage: false, file: null });
    },
    addRight(qIndex) { this.exercises[qIndex].pairs.right.push({ text: "", isImage: false, file: null }); },
    removeLeft(qIndex, lIndex) {
      const q = this.exercises[qIndex];
      q.pairs.left.splice(lIndex, 1);
      delete q.pairs.correctMap[lIndex];
      const newMap = {};
      q.pairs.left.forEach((_, index) => { newMap[index] = q.pairs.correctMap[index] ?? 0; });
      q.pairs.correctMap = newMap;
    },
    removeRight(qIndex, rIndex) {
      const q = this.exercises[qIndex];
      q.pairs.right.splice(rIndex, 1);
      for (const key in q.pairs.correctMap) {
        if (q.pairs.correctMap[key] === rIndex) q.pairs.correctMap[key] = 0;
        else if (q.pairs.correctMap[key] > rIndex) q.pairs.correctMap[key]--;
      }
    },
    onPairLeftImageChange(event, qIndex, lIndex) {
      const file = event.target.files[0];
      const left = this.exercises[qIndex].pairs.left[lIndex];
      left.file = file || null;
      left.preview = file ? URL.createObjectURL(file) : null;
    },
    onPairRightImageChange(event, qIndex, rIndex) {
      const file = event.target.files[0];
      const right = this.exercises[qIndex].pairs.right[rIndex];
      right.file = file || null;
      right.preview = file ? URL.createObjectURL(file) : null;
    },
    async createTest() {
      if (!this.title || this.exercises.length === 0) { this.message = "Тест має містити назву та хоча б одне питання!"; return; }

      try {
        const formData = new FormData();
        formData.append("title", this.title);

        const exercisesData = this.exercises.map(q => {
          const answers = (q.answers || []).map((a, i) => ({
            text: a.isImage ? undefined : a.text || "",
            correct: q.type === "one" ? i === q.correctAnswerIndex : !!a.correct,
            isImage: !!a.isImage
          }));
          const pairs = q.type === "pair" ? { left: q.pairs.left || [], right: q.pairs.right || [], correctMap: q.pairs.correctMap || {} } : null;
          return { type: q.type, question: q.question || "", answers, correctAnswers: q.correctAnswers || [], pairs };
        });
        formData.append("exercises", JSON.stringify(exercisesData));

        this.exercises.forEach((q, qIndex) => {
          (q.answers || []).forEach((a, aIndex) => { if (a.isImage && a.file) formData.append(`images[q${qIndex}][a${aIndex}]`, a.file); });
          if (q.type === "pair") {
            (q.pairs.right || []).forEach((r, rIndex) => { if (r.isImage && r.file) formData.append(`pairImages[q${qIndex}][r${rIndex}]`, r.file); });
            (q.pairs.left || []).forEach((l, lIndex) => { if (l.isImage && l.file) formData.append(`pairImages[q${qIndex}][l${lIndex}]`, l.file); });
          }
        });

        const res = await axios.post(`${BACK_URL}/test`, formData, {
          headers: { Authorization: `Bearer ${localStorage.getItem("tokenAuthTeacher")}`, "Content-Type": "multipart/form-data" }
        });

        this.$root.showToast('Тест створено');
        this.message = '';
        // Зберігаємо код та посилання
        this.testCode = res.data.id;
        this.testLink = window.location.origin + window.location.pathname + `/#/test/${res.data.id}`


        this.showModal = true;

      } catch (err) {
        console.error("AxiosError", err);
        this.message = "Помилка при створенні тесту: " + (err.response?.data?.message || err.message);
        this.$root.showToast("Помилка при створенні тесту", "error")
      }
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto+Slab:wght@100..900&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Roboto Slab', serif;
}

.test-wrapper {
  background-color: #f5f6f8;
  width: 95%;
  max-width: 1000px;
  margin: 30px auto;
  border-radius: 25px;
  padding: 25px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
  transition: all 0.3s;
}

/* HEADER */
.test-header {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 25px;
  padding: 15px 20px;
  border-radius: 20px;
  background: linear-gradient(135deg, #4d0cff, #b000f8, #ff00b3);
  color: white;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);

  /* Додаємо закріплення */
  position: sticky;
  top: 0;
  z-index: 100;
  /* щоб хедер був поверх всього */
}

.test-title {
  font-size: 24px;
  font-weight: 700;
  text-shadow: 1px 1px 4px rgba(0, 0, 0, 0.4);
  flex: 1;
  text-align: center;
}

.back-btn {
  border: none;
  cursor: pointer;
  padding: 8px 18px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.3);
  color: white;
  font-weight: 600;
  transition: 0.3s;
}

.back-btn:hover {
  background: rgba(255, 255, 255, 0.45);
  transform: scale(1.05);
}


/* INPUT / SELECT */
input,
select {
  width: 100%;
  padding: 12px 14px;
  margin-top: 12px;
  border-radius: 12px;
  border: 1px solid #ccc;
  font-size: 16px;
  transition: all 0.3s;
  background-color: #fff;
}

input:focus,
select:focus {
  outline: none;
  border: 2px solid #b000f8;
}

/* BUTTONS */
.btn-remove-question {
  background: linear-gradient(135deg, #ff0000, #ff4d4d);
  margin-top: 10px;
}

.btn-remove-question:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 15px rgba(255, 0, 0, 0.5);
}

.btn-small,
.btn-add-question,
.btn-create-test {
  padding: 10px 16px;
  border-radius: 12px;
  border: none;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s;
  color: white;
  background: linear-gradient(135deg, #4d0cff, #b000f8, #ff00b3);
}

.btn-small:hover,
.btn-add-question:hover,
.btn-create-test:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 15px rgba(255, 0, 179, 0.5);
}

.btn-add {
  margin-top: 10px;
}

/* Відступ перед кнопками внизу */
.btn-add-question,
.btn-create-test {
  padding: 12px;
  font-size: 16px;
  border-radius: 12px;
  border: none;
  cursor: pointer;
  margin-top: 25px;
  background: linear-gradient(135deg, #4d0cff, #b000f8, #ff00b3);
  color: white;
  transition: 0.3s;
}

.btn-down {
  justify-content: center;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}


/* ANSWERS / PAIRS / ENTER */
.answer-row,
.pair-row,
.enter-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 10px;
}

.pairs-block .pair-columns {
  display: flex;
  gap: 20px;
  /* відстань між колонками */
  flex-wrap: wrap;
  /* щоб адаптивно переносило на маленьких екранах */
}

.pairs-block .pair-columns>div {
  flex: 1;
  /* кожна колонка займає половину ширини */
  min-width: 250px;
  /* мінімальна ширина для адаптивності */
}


/* IMAGE PREVIEW */
.preview-img {
  max-width: 120px;
  height: auto;
  border-radius: 12px;
  object-fit: cover;
  margin-top: 5px;
  border: 2px solid rgba(77, 12, 255, 0.6);
  cursor: pointer;
  transition: transform 0.3s, box-shadow 0.3s;
}

.preview-img:hover {
  transform: scale(1.05);
  box-shadow: 0 0 15px rgba(77, 12, 255, 0.5);
}

/* UPLOAD BUTTON */
.upload-label {
  display: inline-block;
  padding: 10px 14px;
  border-radius: 12px;
  background: linear-gradient(135deg, #4d0cff, #b000f8, #ff00b3);
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.3s, box-shadow 0.3s;
  margin-top: 5px;
}

.upload-label:hover {
  transform: scale(1.05);
  box-shadow: 0 0 10px rgba(255, 0, 179, 0.5);
}

.upload-label-input[type="file"] {
  display: none;
}

/* MESSAGES */
.message {
  margin-top: 20px;
  color: rgb(220, 40, 40);
  font-weight: 600;
  font-size: 16px;
}

/* ADAPTIVE */
@media (max-width: 768px) {

  .answer-row,
  .pair-row,
  .enter-row {
    flex-direction: column;
    gap: 8px;
  }

  .test-title {
    font-size: 20px;
  }

  .btn-small,
  .btn-add-question,
  .btn-create-test {
    width: 100%;
  }

  .preview-img {
    max-width: 100%;
  }
}

/*  MODAL WINDOW */
/* Overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 300;
  backdrop-filter: blur(4px);
  animation: fadeIn 0.3s ease forwards;
}

/* Modal Box */
.modal-content {
  background: linear-gradient(135deg, #4d0cff, #b000f8, #ff00b3);
  color: white;
  padding: 50px 40px;
  border-radius: 25px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  max-width: 600px;
  width: 90%;
  text-align: center;
  position: relative;
  transform: scale(0.8);
  animation: popIn 0.3s forwards;
}

/* Animations */
@keyframes popIn {
  to {
    transform: scale(1);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

/* Modal text styles */
.modal-el {
  margin-bottom: 20px;

}

.modal-content h2 {
  margin-bottom: 15px;
  font-size: 26px;
}

.modal-content a {
  color: #fff;
  text-decoration: underline;
  word-break: break-all;
}

.code-text {
  font-weight: 700;
  background: rgba(255, 255, 255, 0.2);
  padding: 2px 8px;
  border-radius: 8px;
  font-family: monospace;
}

/* Copy Button */
.btn-copy {
  margin-left: 10px;
  padding: 5px 12px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 12px;
  border: none;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  transition: all 0.3s;
}

.btn-copy:hover {
  background: rgba(255, 255, 255, 0.4);
  transform: scale(1.1);
}

/* Close Button */
.btn-close {
  position: absolute;
  top: 10px;
  right: 10px;
  border: none;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  font-size: 18px;
  cursor: pointer;
  border-radius: 50%;
  padding: 5px 10px;
  transition: all 0.3s;
}

.btn-close:hover {
  background: rgba(255, 255, 255, 0.4);
  transform: scale(1.2);
}
</style>
