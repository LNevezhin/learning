<template>
  <div class="container" :key="activeCard">
    <div>
      <div class="row">
        <div class="col-6 offset-md-3"><img :src="imageUrl"></div>
      </div>
      <div class="row">
        <div class="col-6 offset-md-4 mt-4">
        </div>
      </div>
      <div>
        <div v-for="(quest, index) in activeQuestCard" :key="index">
          <div class="row mt-4">
            <div class="col-1 offset-md-4">
              <div class="badge badge-pill badge-danger">{{ index + 1}}</div>
            </div>
            <div class="col-3">
              <input :ref="index" :id="index" type="text" class="form-control" v-bind:class="inputStatus[index]"
                v-model.trim="inputData[index]" v-bind:disabled="inputStatus[index] == 'is-valid'"
                @input="checkInput(index)" @focus="getFocus(index)" @blur="lostFocus(index)" autocomplete="off"
                :tabindex="index">
              <div class="valid-feedback">
                Верно!
              </div>
              <div class="invalid-feedback">
                Введите правильный ответ!
              </div>
            </div>
            <div class="col">
              <button type="button" class="btn btn-secondary" v-bind:class="{'sr-only':hintStatus[index] == false}"
                @click=" giveLetter(index)">?</button>
            </div>

          </div>
        </div>
        <div class="col-1 mb-4 offset-md-5 mt-3"><button id="next" type="button" class="btn btn-secondary"
            v-bind:class="{'sr-only':nextStatusButton == true}" @click="getNextCard()">Далее</button></div>
      </div>

    </div>
  </div>
</template>

<script>
  export default {
    name: 'quest',
    data() {
      return {
        activeUrl: '',
        activeQuestCard: [],
        inputData: [],
        correctInputLetters: [], // массив с правильно введенными буквами
        inputStatus: [],
        indexInputLetters: [], // индекс текущей буквы в correctInputLetters[]
        hintStatus: [],
        errCount: [],
        delayStopInput: 5000,
        delayFocusLost: 3000,
        nextStatusButton: true,
        timerStopInput: [],
        timerFocusLost: [],
        activeCard: 3, // номер текущей карточки, оно же ключ, для обновления ДОМ
        ajaxApi: [["0", "pic0000.png", "ПоМиДоР", "КаПуСтА", "пЕрЕц", "БаКЛАЖан", "ЧЕСНОК"],
        ["1", "pic0001.png", "Москва", "Киев", "Минск", "Баку", "Ереван", "Тбилиси", "Алматы"],
        ["2", "pic0002.png", "КруГ", "КвадраТ", "ТреугольниК", "РомБ", "ТрапециЯ", "ЭллипС"],
        ["3", "pic0003.png", "красныЙ", "белыЙ", "черныЙ"]]
      }
    },

    created() {
      if (sessionStorage.getItem('activeCard') != null)
        this.activeCard = sessionStorage.getItem('activeCard');

      this.activeUrl = this.ajaxApi[this.activeCard][1];
      for (let i = 0; i < this.ajaxApi[this.activeCard].length - 2; i++) {
        this.activeQuestCard[i] = this.ajaxApi[this.activeCard][i + 2].toLowerCase();
        this.hintStatus[i] = false;
        this.indexInputLetters[i] = 0;
        this.inputStatus[i] = '';
        this.errCount[i] = 0;
        this.inputData[i] = '';
        this.correctInputLetters[i] = '';
        this.timerStopInput[i] = 0;
        this.timerFocusLost[i] = 0;
      }
    },

    computed: {
      imageUrl() {
        return require(`@/assets/img/${this.activeUrl}`)
      }
    },

    methods: {
      checkInput: function (index) {
        this.inputData[index] = this.inputData[index].toLowerCase();
        this.indexInputLetters[index] = this.inputData[index].length - 1;

        if (this.inputData[index][this.indexInputLetters[index]] != this.activeQuestCard[index][this.indexInputLetters[index]]) {
          this.inputStatus[index] = 'is-invalid';
          this.inputData[index] = this.correctInputLetters[index];
          this.errCount[index]++;
        }
        else {
          this.inputStatus[index] = '';
          this.errCount[index] = 0;
          this.correctInputLetters[index] = this.inputData[index];
        }
        if (this.inputData[index] == this.activeQuestCard[index].toLowerCase()) {
          this.inputStatus[index] = 'is-valid';
          this.setFocus();
        }
        if (this.errCount[index] == 3) {
          this.hintStatus[index] = 'true';
        }
        this.stopInput(index);
        this.nextStatusButton = !this.inputStatus.every(elem => elem == 'is-valid')
      },

      stopInput: function (index) {
        clearTimeout(this.timerStopInput[index]);
        this.timerStopInput[index] = setTimeout(() => {
          if (this.inputStatus[index] != 'is-valid') {
            this.inputStatus[index] = 'is-invalid'
            this.$forceUpdate();
          }
        }, this.delayStopInput)
      },

      getFocus: function (index) {
        this.stopInput(index);
        this.inputStatus[index] = '';
        clearTimeout(this.timerFocusLost[index]);
        this.$forceUpdate();
      },

      lostFocus: function (index) {
        this.timerFocusLost[index] = setTimeout(() => {
          if (this.inputStatus[index] != 'is-valid') {
            this.inputStatus[index] = 'is-invalid'
          }
        }, this.delayFocusLost)
        this.$forceUpdate();
      },

      giveLetter: function (index) {
        this.inputData[index] = this.inputData[index] + this.activeQuestCard[index][this.indexInputLetters[index]];
        this.$forceUpdate();
        this.inputStatus[index] = '';
        this.correctInputLetters[index] = this.inputData[index];
        this.indexInputLetters[index] = this.indexInputLetters[index] + 1;
        if (this.inputData[index] == this.activeQuestCard[index].toLowerCase()) {
          this.inputStatus[index] = 'is-valid';
          this.hintStatus[index] = false;
          this.setFocus();
        }
        this.nextStatusButton = !this.inputStatus.every(elem => elem == 'is-valid')
      },

      setFocus: function () {
        const id = this.inputStatus.findIndex(elem => elem != 'is-valid')
        console.log('id: ', id);

        if (id != -1) {
          const input = document.getElementById(id);
          input.focus()
        }
        else {
          const input = document.getElementById('next')
          input.focus()
        }
      },

      getNextCard: function () {
        this.activeCard = parseInt(this.activeCard) + 1;
        if (this.activeCard == 4) this.activeCard = 0;
        sessionStorage.setItem('activeCard', this.activeCard);
        this.nextStatusButton = true;
        location.reload()
      }
      /* Этот говно код не заработал, поэтому ппришлось еще говнее придумать:) */
      /*getNextCard: function () {
        this.activeCard = parseInt(this.activeCard) + 1;
        this.$nextTick(() => { this.$forceUpdate(); })
      } */
    }
  }
</script>

<style scoped>
  div {
    user-select: none;
  }
</style>