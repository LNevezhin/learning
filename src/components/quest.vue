<template>
  <div class="container" :key="activeCard">
    <div>
      <div class="row">
        <div class="col-6 offset-3"><img :src="imageUrl" /></div>
      </div>
      <div class="row">
        <div class="col-6 offset-4 mt-4"></div>
      </div>
      <div>
        <div v-for="(quest, index) in activeQuestCard" :key="parseInt(index)">
          <div class="row mt-4">
            <div class="col-1 offset-4">
              <div class="badge badge-pill badge-danger">{{ index + 1 }}</div>
            </div>
            <div class="col-3">
              <input
                :ref="parseInt(index)"
                :id="parseInt(index)"
                type="text"
                class="form-control"
                v-bind:class="inputStatus[index]"
                v-model.trim="inputData[index]"
                v-bind:disabled="inputStatus[index] == 'is-valid'"
                @input="checkInput(index)"
                @focus="getFocus(index)"
                @blur="lostFocus(index)"
                autocomplete="off"
              />
              <div class="valid-feedback">
                Верно!
              </div>
              <div class="invalid-feedback">
                Введите правильный ответ!
              </div>
            </div>
            <div class="col">
              <button
                type="button"
                class="btn btn-secondary"
                v-bind:class="{ 'sr-only': hintStatus[index] == false }"
                @click="giveLetter(index)"
              >
                ?
              </button>
            </div>
          </div>
        </div>
        <div class="col-1 mb-4 offset-5 mt-3">
          <button
            id="next"
            type="button"
            class="btn btn-secondary"
            v-bind:class="{ 'sr-only': nextStatusButton == true }"
            @click="getNextCard()"
          >
            Далее
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "quest",
  data() {
    return {
      activeUrl: "",
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
      activeCard: 0, // номер текущей карточки, оно же ключ, для обновления ДОМ
      ajaxApi: [
        [
          "0",
          "pic0000.png",
          "ПоМиДоР",
          "КаПуСтА",
          "пЕрЕц",
          "БаКЛАЖан",
          "ЧЕСНОК",
        ],
        [
          "1",
          "pic0001.png",
          "Москва",
          "Киев",
          "Минск",
          "Баку",
          "Ереван",
          "Тбилиси",
          "Алматы",
        ],
        ["2", "pic0003.png", "красныЙ", "белыЙ", "черныЙ"],
        [
          "3",
          "pic0002.png",
          "КруГ",
          "КвадраТ",
          "ТреугольниК",
          "РомБ",
          "ТрапециЯ",
          "ЭллипС",
        ],
      ],
    };
  },

  created() {
    if (sessionStorage.getItem("activeCard") != null)
      this.activeCard = sessionStorage.getItem("activeCard");

    this.activeUrl = this.ajaxApi[this.activeCard][1];
    for (let i = 0; i < this.ajaxApi[this.activeCard].length - 2; i++) {
      this.activeQuestCard[i] = this.ajaxApi[this.activeCard][
        i + 2
      ].toLowerCase();
      this.hintStatus[i] = false;
      this.indexInputLetters[i] = 0;
      this.inputStatus[i] = "";
      this.errCount[i] = 0;
      this.inputData[i] = "";
      this.correctInputLetters[i] = "";
      this.timerStopInput[i] = 0;
      this.timerFocusLost[i] = 0;
    }
  },

  computed: {
    imageUrl() {
      return require(`@/assets/img/${this.activeUrl}`);
    },
  },

  mounted: function() {
    this.setFocus();
  },

  methods: {
    checkInput: function(index) {
      this.stopInput(index);
      this.inputData[index] = this.inputData[index].toLowerCase();
      this.indexInputLetters[index] = this.inputData[index].length - 1;
      if (
        this.inputData[index][this.indexInputLetters[index]] !=
        this.activeQuestCard[index][this.indexInputLetters[index]]
      )
        this.incorrectInput(index);
      else this.correctInput(index);
    },

    correctInput: function(index) {
      this.setInputStatus("");
      this.errCount[index] = 0;
      this.correctInputLetters[index] = this.inputData[index];
      this.validCheck(index);
    },

    validCheck: function(index) {
      if (this.inputData[index] == this.activeQuestCard[index]) {
        this.setInputStatus(index, "is-valid");
        this.hintStatus[index] = false;
        this.setFocus();
      }
    },

    incorrectInput: function(index) {
      this.setInputStatus(index, "is-invalid");
      this.inputData[index] = this.correctInputLetters[index];
      this.errCount[index]++;
      if (this.errCount[index] == 3) this.hintStatus[index] = "true";
    },

    setInputStatus: function(index, status) {
      this.inputStatus[index] = status;
    },

    stopInput: function(index) {
      clearTimeout(this.timerStopInput[index]);
      this.timerStopInput[index] = setTimeout(() => {
        if (this.inputStatus[index] != "is-valid") {
          this.setInputStatus(index, "is-invalid");
          this.$forceUpdate();
        }
      }, this.delayStopInput);
    },

    getFocus: function(index) {
      this.stopInput(index);
      this.setInputStatus(index, "");
      clearTimeout(this.timerFocusLost[index]);
      this.$forceUpdate();
    },

    lostFocus: function(index) {
      this.timerFocusLost[index] = setTimeout(() => {
        if (this.inputStatus[index] != "is-valid")
          this.setInputStatus(index, "is-invalid");
      }, this.delayFocusLost);
      this.$forceUpdate();
    },

    giveLetter: function(index) {
      this.setFocus(index);
      this.inputData[index] =
        this.inputData[index] +
        this.activeQuestCard[index][this.indexInputLetters[index]];
      this.$forceUpdate();
      this.correctInputLetters[index] = this.inputData[index];
      this.indexInputLetters[index] = this.indexInputLetters[index] + 1;
      this.validCheck(index);
    },

    setFocus: function(item) {
      const id = this.inputStatus.findIndex((elem) => elem != "is-valid");
      if (id != -1 && this.inputStatus[item] != "is-valid" && item != null) {
        const input = document.getElementById(item);
        input.focus();
      }
      if (id != -1 && item == null) {
        const input = document.getElementById(id);
        input.focus();
      }
      if (id == -1) {
        this.nextStatusButton = false;
        const input = document.getElementById("next");
        input.focus();
      }
    },

    getNextCard: function() {
      this.activeCard = parseInt(this.activeCard) + 1;
      if (this.activeCard == this.ajaxApi.length) this.activeCard = 0;
      sessionStorage.setItem("activeCard", this.activeCard);
      this.nextStatusButton = true;
      location.reload();
    },
  },
};
</script>

<style scoped>
div {
  user-select: none;
}
</style>
