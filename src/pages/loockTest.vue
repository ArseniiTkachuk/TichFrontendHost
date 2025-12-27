<template>
    <div v-if="test.exercises" class="test-view-container">
        <nav class="navigation">
            <button @click="$router.back()" class="btn-back">
                <div class="blur-bg"></div>
                <span class="arrow">←</span>
                <span class="btn-text">Назад</span>
            </button>

            <button class="btn-share" title="Поділитися тестом" @click="showModal = true">
                <div class="blur-bg"></div>
                <span class="icon">🔗</span>
                <span class="btn-text">Поділитися</span>
            </button>
        </nav>

        <header class="test-header">
            <div class="header-content">
                <h1>{{ test.title }}</h1>
                <div class="test-stats">
                    <span class="stat-item">
                        <strong>{{ test.childrens.length }}</strong> проходжень
                    </span>
                    <span class="divider">|</span>
                    <span class="stat-item">
                        <strong>{{ test.exercises.length }}</strong> завдань
                    </span>
                </div>
            </div>
        </header>

        <main class="exercises-list">
            <div v-for="(exercise, index) in test.exercises" :key="exercise.slug || index" class="exercise-card">
                <div class="exercise-badge">Завдання №{{ index + 1 }}</div>

                <div class="exercise-body">
                    <answers v-if="exercise.type === 'one' || exercise.type === 'many'" :ex="exercise" />

                    <enters v-if="exercise.type === 'enter'" :ex="exercise" />

                    <pairs v-if="exercise.type === 'pair'" :ex="exercise" />
                </div>
            </div>
        </main>
    </div>
    <div v-else class="loading-state">
        <p>Завантаження тесту...</p>
    </div>

    <!-- Модальне вікно поділитися тестом -->
    <div v-if="showModal" class="modal-overlay" @click="showModal = false">
        <div class="modal-content animated-modal" @click.stop>
            <h2>🔗 Поділитися тестом!</h2>
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
        <div v-if="copied" class="copied-toast">Скопійовано!</div>
    </div>

</template>

<script>
import axios from 'axios';

import answers from '@/components/loockTest/answers.vue';
import enters from '@/components/loockTest/enters.vue';
import pairs from '@/components/loockTest/pairs.vue';
const BackURL = "https://test-bg-prj-for-math.onrender.com";



export default {
    components: {
        answers,
        enters,
        pairs
    },
    data() {
        return {
            test: {},
            showModal: false,
            testCode: "",
            testLink: "",
            copied: false,
        };
    },

    mounted() {
        this.fetchTest();
    },

    methods: {
        async fetchTest() {
            try {
                const testId = this.$route.params.id;
                const res = await axios.get(`${BackURL}/getOneTest/${testId}`);
                this.test = res.data.test;

                // Зберігаємо код та посилання
                this.testCode = testId;
                this.testLink = `${window.location.origin}/test/${testId}`;

            } catch (err) {
                console.error(err);
            }
        },

        copyToClipboard(text) {
            navigator.clipboard.writeText(text).then(() => {
                this.copied = true;
                setTimeout(() => (this.copied = false), 1500);
            }).catch(err => console.error(err));
        },
    }
};
</script>

<style scoped>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Roboto Slab', serif;
}


:global(body) {
    background: linear-gradient(135deg, #4d0cff, #b000f8, #ff00b3);
    background-size: 300% 300%;
    animation: gradientMove 10s ease infinite;
}


/* Анімація градієнта */
@keyframes gradientMove {
    0% {
        background-position: 0% 50%;
    }

    50% {
        background-position: 100% 50%;
    }

    100% {
        background-position: 0% 50%;
    }
}




.test-view-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    position: relative;
    z-index: 1;
    /* щоб бути над анімацією body */
}

.navigation {
    margin-bottom: 30px;
    display: flex;
    justify-content: space-between;
    /* Розносить кнопки в різні боки */
    align-items: center;
    gap: 15px;
}

/* Базові стилі для обох кнопок (можна згрупувати) */
.btn-back,
.btn-share {
    position: relative;
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 10px 20px;
    background: rgba(255, 255, 255, 0.15);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-radius: 14px;
    color: #ffffff;
    font-weight: 600;
    font-size: 15px;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
    overflow: hidden;
}

/* Специфічні стилі для кнопки Share */
.btn-share {
    background: rgba(255, 255, 255, 0.1);
    border-color: rgba(255, 255, 255, 0.2);
}

.btn-share:hover {
    background: rgba(77, 12, 255, 0.3);
    /* Додаємо легкий колір бренду при наведенні */
    border-color: rgba(255, 255, 255, 0.5);
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(77, 12, 255, 0.2);
}

.icon {
    font-size: 18px;
}

/* Анімація для іконки share при наведенні */
.btn-share:hover .icon {
    transform: rotate(-15deg) scale(1.1);
    transition: transform 0.3s ease;
}

/* Адаптивність: на малих екранах ховаємо текст, залишаємо іконки */
@media (max-width: 480px) {
    .btn-text {
        display: none;
    }

    .btn-back,
    .btn-share {
        padding: 10px 15px;
    }
}

.btn-back:hover {
    background: rgba(255, 255, 255, 0.25);
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
    border-color: rgba(255, 255, 255, 0.5);
}

.btn-back:active {
    transform: translateY(0);
}

.arrow {
    font-size: 20px;
    transition: transform 0.3s ease;
}

.btn-back:hover .arrow {
    transform: translateX(-5px);
    /* Стрілочка рухається вліво при наведенні */
}

/* Шапка */
.test-header {
    background: linear-gradient(135deg, #4d0cff 0%, #b000f8 100%);
    color: white;
    padding: 35px;
    border-radius: 24px;
    margin-bottom: 30px;
    box-shadow: 0 10px 25px rgba(176, 0, 248, 0.2);
}

.test-header h1 {
    margin: 0 0 10px 0;
    font-size: 26px;
}

.test-stats {
    display: flex;
    align-items: center;
    gap: 15px;
    font-size: 15px;
    opacity: 0.9;
}

.divider {
    opacity: 0.5;
}


/* Надпис завантаження тесту */
.loading-state {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    color: white;
    font-size: 24px;
    font-weight: bold;
    text-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
}

/* Картки завдань */
.exercise-card {
    background: white;
    border-radius: 20px;
    padding: 25px;
    margin-bottom: 25px;
    border: 1px solid #eaeaea;
}

.exercise-badge {
    display: inline-block;
    padding: 4px 12px;
    background: #f3e5f5;
    color: #b000f8;
    border-radius: 8px;
    font-size: 12px;
    font-weight: 700;
    margin-bottom: 20px;
}

/* Тип Enter */
.question {
    font-size: 20px;
    margin-bottom: 20px;
}

.answer-info {
    background: #f1f8e9;
    border: 1px solid #c5e1a5;
    padding: 15px;
    border-radius: 12px;
}

.label {
    display: block;
    font-size: 13px;
    color: #558b2f;
    margin-bottom: 8px;
    font-weight: 600;
}

.chips {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
}

.ans-chip {
    background: #fff;
    padding: 4px 12px;
    border-radius: 6px;
    color: #2e7d32;
    font-weight: 700;
    border: 1px solid #c5e1a5;
}

.placeholder-box {
    padding: 30px;
    border: 2px dashed #ddd;
    border-radius: 15px;
    text-align: center;
    color: #999;
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

/* toast notification */
.copied-toast {
    position: fixed;
    bottom: 10%;
    right: 0%;
    transform: translateX(-50%);
    background: #4caf50;
    color: white;
    padding: 15px 40px;
    /* Збільшуємо розмір блоку */
    border-radius: 20px;
    /* Більші закруглення */
    font-size: 20px;
    /* Збільшуємо текст */
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
    animation: slideInOut 1.5s forwards;
    z-index: 500;
}

@keyframes slideInOut {
    0% {
        opacity: 0;
        transform: translateX(-50%) translateY(20px);
    }

    10%,
    90% {
        opacity: 1;
        transform: translateX(-50%) translateY(0);
    }

    100% {
        opacity: 0;
        transform: translateX(-50%) translateY(20px);
    }
}
</style>