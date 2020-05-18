<template>
  <div id="app">
    <div class="container">
      <h5 class="title is-4">Васильев Г.А, гр. 4940М</h5>
      <section class="section">
        <h5 class="title is-5">Обучение</h5>
        <h6 class="title is-6">Идеальные образцы</h6>
        <div class="flex flex-center">
          <img :src="letters.perfect.B.image" />
          <img :src="letters.perfect.G.image" />
          <img :src="letters.perfect.A.image" />
          <img :src="letters.perfect.once.image" />
        </div>
        <div class="flex flex-column flex-center mt-20">
          <b-field label="Макс. кол-во итераций обучения">
            <b-numberinput v-model="trainConfig.iterations" />
          </b-field>
          <b-field label="Допустимый процент ошибок">
            <b-numberinput
              v-model="trainConfig.errorThresh"
              :min="0"
              :step="0.001"
            />
          </b-field>
          <div class="field mt-20">
            <b-switch v-model="trainConfig.log">
              Логировать ли обучение (в консоль)
            </b-switch>
          </div>
          <b-field label="Шаг логирования">
            <b-numberinput
              v-model="trainConfig.logPeriod"
              :disabled="!trainConfig.log"
            />
          </b-field>
          <b-field label="Скорость обучения">
            <b-numberinput
              v-model="trainConfig.learningRate"
              :step="0.05"
              :min="0"
              :max="1"
            />
          </b-field>
          <b-button type="is-primary" @click="train">
            Запустить обучение
          </b-button>
          <b-button
            v-if="trainStatus"
            type="is-danger"
            @click="reset"
            class="mt-20"
          >
            Сброс обучения
          </b-button>
        </div>
      </section>
      <section class="section">
        <h5 class="title is-5">Тренировка</h5>
        <div class="flex flex-center">
          <b-radio
            v-for="(v, i) in letters.notPerfect.B"
            :key="`B-${i}`"
            v-model="currLetter"
            name="currLetter"
            :native-value="v.value"
            :disabled="!trainStatus"
          >
            <img :src="v.image" />
          </b-radio>
          <b-radio
            v-for="(v, i) in letters.notPerfect.G"
            :key="`G-${i}`"
            v-model="currLetter"
            name="currLetter"
            :native-value="v.value"
            :disabled="!trainStatus"
          >
            <img :src="v.image" />
          </b-radio>
          <b-radio
            v-for="(v, i) in letters.notPerfect.A"
            :key="`A-${i}`"
            v-model="currLetter"
            name="currLetter"
            :native-value="v.value"
            :disabled="!trainStatus"
          >
            <img :src="v.image" />
          </b-radio>
          <b-radio
            v-model="currLetter"
            name="currLetter"
            :native-value="letters.notPerfect.once.value"
            :disabled="!trainStatus"
          >
            <img :src="letters.notPerfect.once.image" />
          </b-radio>
          <b-radio
            v-model="currLetter"
            name="currLetter"
            :native-value="letters.unstudied.value"
            :disabled="!trainStatus"
          >
            <img :src="letters.unstudied.image" />
          </b-radio>
        </div>
        <div class="flex flex-center">
          <b-button
            type="is-primary"
            :disabled="!trainStatus || !currLetter"
            @click="check"
          >
            Тренировать
          </b-button>
        </div>
      </section>
      <section v-if="result" class="section">
        <h5 class="title is-5">Результат</h5>
        <div v-for="(v, i) in result" :key="i" class="res">
          {{ i }} - {{ v }}
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import brain from 'brain.js'
import {
  perfectLetters,
  notPerfectLetters,
  unstudiedLetter,
} from './characters'

const config = {
  binaryThresh: 0.5,
  hiddenLayers: [3],
  activation: 'sigmoid',
  leakyReluAlpha: 0.01,
}

export default {
  name: 'App',
  components: {},
  data: () => ({
    // объет нейронной сети
    net: new brain.NeuralNetwork(config),
    // массив объектов для обучения и тренировки
    letters: {
      perfect: perfectLetters, // идеальные символы
      notPerfect: notPerfectLetters, // не идеальные символы
      unstudied: unstudiedLetter, // случайный символ
    },
    // конфиг обучения
    trainConfig: {
      iterations: 1000, // макс. кол-во итераций обучения
      errorThresh: 0.005, // допустимый процент ошибок обучения, от 0 до 1
      log: false, // нужно ли производить логирование (в консоль)
      logPeriod: 10, // шаг итерации для записи лога
      learningRate: 0.3, // скорость обучения, от 0 до 1
      timeout: Infinity, // максимальное количество миллисекунд для обучения
    },
    trainStatus: false, // статус обучения
    currLetter: null, // выбранный символ для тренировки (реактивный объект)
    result: null, // выбранный символ для тренировки (реактивный объект)
  }),
  methods: {
    /*
      !!Обучение на основе идеальных данных!!

      net - объект нейронной сети
      letters.perfect = perfectLetters
      trainConfig = конфигурация обучения
    */
    async train() {
      // запуск процесса обучения
      const res = await this.net.train(
        [
          { input: this.letters.perfect.B.value, output: { 'В': 1 } },
          { input: this.letters.perfect.G.value, output: { 'Г': 1 } },
          { input: this.letters.perfect.A.value, output: { 'А': 1 } },
          { input: this.letters.perfect.once.value, output: { '1': 1 } },
        ],
        this.trainConfig,
      )

      // если результат успешен
      if (res) {
        // вывод оповещения + информация о проценте ошибок и кол-во итераций
        this.$buefy.snackbar.open({
          message: `Обучение успешно завершено<br />Процент ошибок: ${res.error}<br /> Количество интераций: ${res.iterations}`,
          position: 'is-bottom',
          actionText: 'Закрыть',
          indefinite: true,
        })
        // если включен дебаг, то вывести в консоль результирующие весовые коэффициенты
        if (this.trainConfig.log)
          console.log('result weights:', this.net.weights)
        // установить статус завершения обучения
        this.trainStatus = true
      }
    },
    /*
      !!Сброс обучения!!
      Реинициализация нейронной сети
      Сброс резульата
      Сброс статуса обучения
    */
    reset() {
      this.net = new brain.NeuralNetwork(config)
      this.result = null
      this.trainStatus = false
    },
    /*
      !!Тренировка на основе выбранной буквы!!

      net - объект нейронной сети
      currLetter = выбранная буква
      result = { ...буква: вероятность совпадения }
    */
    check() {
      this.result = this.net.run(this.currLetter)
      this.$buefy.snackbar.open({
        message: 'Тренировка успешно завершена',
        position: 'is-bottom',
        actionText: 'Закрыть',
      })
    },
  },
}
</script>

<style>
#app {
  margin: 20px 0;
}

.title,
.label,
.res {
  text-align: center;
}

.flex {
  display: flex;
}
.flex-center {
  justify-content: center;
  align-items: center;
}

.flex-column {
  flex-direction: column;
}

.input {
  max-width: 250px;
}

.mt-20 {
  margin-top: 20px;
}
</style>
