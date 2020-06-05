<template>
  <div class="container" :key="activeCard">
    <div>
      <div id="empty">пустота</div>
      <div v-for="(quest, index) in activeQuestCard" :key="parseInt(index)">
        <div class="row align-items-center">
          <div class="col-1 offset-4">
            <div class="badge badge-pill badge-danger">{{ index + 1 }}</div>
          </div>
          <div id="country" class="col">{{randomCountry[index]}}</div>
        </div>
      </div>
      <div>
        <div id="empty">пустота</div>
        <div class="row mt-4" v-for="(quest, index) in activeQuestCard" :key="parseInt(index)">
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
            <div class="valid-feedback">Верно!</div>
            <div class="invalid-feedback">Введите правильный ответ!</div>
          </div>
          <div class="col">
            <button
              type="button"
              class="btn btn-secondary"
              v-bind:class="{ 'sr-only': hintStatus[index] == false }"
              @click="giveLetter(index)"
            >?</button>
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
        >Далее</button>
      </div>
    </div>
    <div id="empty">пустота</div>
    <!--     <div id="count">{{this.letterCount}}</div> -->
  </div>
</template>

<script>
export default {
  name: "quest",
  data() {
    return {
      randomCountry: [],
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
      activeCard: 0,
      totalLetters: 0,
      letterCount: 0,
      randomCountryIndex: [], // номер текущей карточки, оно же ключ, для обновления ДОМ
      ajaxApi: [
        ["Андорра", "Андорра-ла-Велья"],
        ["ОАЭ", "Абу-Даби"],
        ["Афганистан", "Кабул"],
        ["Антигуа и Барбуда", "Сент-Джонс"],
        ["Ангилья", "Валли"],
        ["Албания", "Тирана"],
        ["Армения", "Ереван"],
        ["Ангола", "Луанда"],
        ["Аргентина", "Буэнос-Айрес"],
        ["Американское Самоа", "Пагопаго"],
        ["Австрия", "Вена"],
        ["Австралия", "Канберра"],
        ["Аруба", "Ораньестад"],
        ["Азербайджан", "Баку"],
        ["Босния", "Сараево"],
        ["Барбадос", "Бриджтаун"],
        ["Бангладеш", "Дакка"],
        ["Бельгия", "Брюссель"],
        ["Буркина Фасо", "Уагадугу"],
        ["Болгария", "София"],
        ["Бахрейн", "Манама"],
        ["Бурунди", "Бужумбура"],
        ["Бенин", "Порто-Ново"],
        ["Бермудские о-ва", "Гамильтон"],
        ["Бруней", "Бандар-Сери-Бегаван"],
        ["Боливия", "Сукре"],
        ["Бразилия", "Бразилия"],
        ["Багамы", "Нассау"],
        ["Бутан", "Тхимпху"],
        ["Ботсвана", "Габороне"],
        ["Беларусь", "Минск"],
        ["Белиз", "Бельмопан"],
        ["Канада", "Оттава"],
        ["Демократическая Республика Конго", "Киншаса"],
        ["Центральная Африканская Республика", "Банги"],
        ["Швейцария", "Берн"],
        ["Кот-д'Ивуар", "Ямусукро"],
        ["Острова Кука", "Аваруа"],
        ["Чили", "Сантьяго"],
        ["Камерун", "Яунде"],
        ["Китай", "Пекин"],
        ["Колумбия", "Богота"],
        ["Коста-Рика", "Сан-Хосе"],
        ["Куба", "Гавана"],
        ["Острова Зеленого Мыса", "Прая"],
        ["Кипр", "Никосия"],
        ["Чешская Республика", "Прага"],
        ["Германия", "Берлин"],
        ["Джибути", "Джибути"],
        ["Дания", "Копенгаген"],
        ["Доминиканская Республика", "Санто-Доминго"],
        ["Алжир", "Алжир"],
        ["Эквадор", "Кито"],
        ["Эстония", "Таллин"],
        ["Египет", "Каир"],
        ["Западная Сахара", "Эль-Аюн"],
        ["Эритрея", "Асмэра"],
        ["Испания", "Мадрид"],
        ["Эфиопия", "Аддис-Абеба"],
        ["Финляндия", "Хельсинки"],
        ["Фиджи", "Сува"],
        ["Микронезия", "Паликир"],
        ["Фарерские о-ва", "Торсхавн"],
        ["Франция", "Париж"],
        ["Габон", "Либревиль"],
        ["Великобритания", "Лондон"],
        ["Гренада", "Сент-Джоржес"],
        ["Грузия", "Тбилиси"],
        ["Французская Гвиана", "Кайенна"],
        ["Гернси", "Сент-Питер-Порт"],
        ["Гана", "Аккра"],
        ["Гибралтар", "Гибралтар"],
        ["Гренландия", "Нуук"],
        ["Гамбия", "Банжул"],
        ["Гвинея", "Конакри"],
        ["Гваделупа", "Бас-Тер"],
        ["Экваториальная Гвинея", "Малабо"],
        ["Греция", "Афины"],
        ["Гватемала", "Гватемала"],
        ["Гуам", "Аганья"],
        ["Гвинея-Биссау", "Бисау"],
        ["Гайана", "Джорджтаун"],
        ["Гонконг", "Гонконг"],
        ["Гондурас", "Тегусигальпа"],
        ["Хорватия", "Загреб"],
        ["Гаити", "Порт-о-Пренс"],
        ["Венгрия", "Будапешт"],
        ["Индонезия", "Джакарта"],
        ["Ирландия", "Дублин"],
        ["Мэн, о-в", "Дуглас"],
        ["Индия", "Нью-Дели"],
        ["Ирак", "Багдад"],
        ["Иран", "Тегеран"],
        ["Исландия", "Рейкьявик"],
        ["Италия", "Рим"],
        ["Джерси", "Сент-Хельер"],
        ["Ямайка", "Кингстон"],
        ["Иордания", "Амман"],
        ["Япония", "Токио"],
        ["Кения", "Найроби"],
        ["Кыргызстан", "Бишкек"],
        ["Камбоджа", "Пномпень"],
        ["Кирибати", "Южная Тарава"],
        ["Коморские о-ва", "Морони"],
        ["Сент-Киттс и Невис", "Бастер"],
        ["Северная Корея", "Пхеньян"],
        ["Южная Корея", "Сеул"],
        ["Кувейт", "Кувейт"],
        ["Каймановы Острова", "Джоржтаун"],
        ["Казахстан", "Астана"],
        ["Лаос", "Вьентьян"],
        ["Ливан", "Бейрут"],
        ["Сент-Люсия", "Кастри"],
        ["Лихтенштейн", "Вадуц"],
        ["Шри-Ланка", "Коломбо"],
        ["Либерия", "Монровия"],
        ["Лесото", "Масеру"],
        ["Литва", "ВИЛЬНЮС"],
        ["Люксембург", "Люксембург"],
        ["Латвия", "Рига"],
        ["Ливия", "Триполи"],
        ["Марокко", "Рабат"],
        ["Монако", "Монако"],
        ["Молдова", "Кишинев"],
        ["Черногория", "Подгорица"],
        ["Остров Святого Мартина", "Мариго"],
        ["Мадагаскар", "Антананариву"],
        ["Маршалловы о-ва", "Маджуро"],
        ["Македония", "Скопье"],
        ["Мали", "Бамако"],
        ["Мьянма", "Нейпьидо"],
        ["Монголия", "Улан-Батор"],
        ["Макао", "Макао"],
        ["Северные Марианские о-ва", "Сайпан"],
        ["Мартиника", "Форт-де-Франс"],
        ["Мавритания", "Нуакшот"],
        ["Монсеррат", "Плимут"],
        ["Мальта", "Валлетта"],
        ["Маврикий", "Порт-Луи"],
        ["Мальдивские о-ва", "Мале"],
        ["Малави", "Лилонгве"],
        ["Мексика", "Мехико"],
        ["Малайзия", "Куала-Лумпур"],
        ["Мозамбик", "Мапуту"],
        ["Намибия", "Виндхук"],
        ["Новая Каледония", "Нумеа"],
        ["Нигерия", "Ниамей"],
        ["Остров Норфолк", "Кингстон"],
        ["Нигерия", "Абуджа"],
        ["Никарагуа", "Манагуа"],
        ["Нидерланды", "Амстердам"],
        ["Норвегия", "Осло"],
        ["Непал", "Катманду"],
        ["Новая Зеландия", "Веллингтон"],
        ["Оман", "Маскат"],
        ["Панама", "Панама"],
        ["Перу", "Лима"],
        ["Французская Полинезия", "Папеэте"],
        ["Папуа – Новая Гвинея", "Порт-Морсби"],
        ["Филиппины", "Манила"],
        ["Пакистан", "Исламабад"],
        ["Польша", "Варшава"],
        ["Пуэрто-Рико", "Сан-Хуан"],
        ["Португалия", "Лиссабон"],
        ["Парагвай", "Асунсьон"],
        ["Катар", "Доха"],
        ["Румыния", "Бухарест"],
        ["Сербия", "Белград"],
        ["Россия", "Москва"],
        ["Руанда", "Кигали"],
        ["Саудовская Аравия", "Эр-Рияд"],
        ["Соломонские острова", "Хониара"],
        ["Сейшельские о-ва", "Виктория"],
        ["Судан", "Хартум"],
        ["Швеция", "Стокгольм"],
        ["Сингапур", "Сингапур"],
        ["Остров Святой Елены", "Джеймстаун"],
        ["Словения", "Любляна"],
        ["Словакия", "Братислава"],
        ["Сьерра-Леоне", "Фритаун"],
        ["Сан-Марино", "Сан-Марино"],
        ["Сенегал", "Дакар"],
        ["Сомали", "Могадишу"],
        ["Суринам", "Парамарибо"],
        ["Южный Судан", "Джуба"],
        ["Сан-Томе и Принсипе", "Сан-Томе"],
        ["Сальвадор", "Сан-Сальвадор"],
        ["Сирийская Арабская Респ.", "Дамаск"],
        ["Свазиленд", "Мбабане"],
        ["Чад", "Нджамена"],
        ["Того", "Ломе"],
        ["Таиланд", "Бангкок"],
        ["Таджикистан", "Душанбе"],
        ["Восточный Тимор", "Дили"],
        ["Туркменистан", "Ашхабад"],
        ["Тунис", "Тунис"],
        ["Тонга", "Нукуалофа"],
        ["Турция", "Анкара"],
        ["Тринидад и Тобаго", "Порт-оф-Спейн"],
        ["Тувалу", "Funafuti"],
        ["Тайвань", "Тайбэй"],
        ["Танзания", "Додома"],
        ["Украина", "Киев"],
        ["Уганда", "Кампала"],
        ["США", "Вашингтон"],
        ["Уругвай", "Монтевидео"],
        ["Узбекистан", "Ташкент"],
        ["Ватикан", "Ватикан"],
        ["Сент-Винсент и Гренадины", "Кингстаун"],
        ["Венесуэла", "Каракас"],
        ["Британские Виргинские о-ва", "Роадтаун"],
        ["Вьетнам", "Ханой"],
        ["Вануату", "Порт-Вила"],
        ["Уоллис и Футуна", "Мата-Уту"],
        ["Самоа", "Апиа"],
        ["Йемен", "Сана"],
        ["Майотта", "Мамудзу"],
        ["ЮАР", "Претория"],
        ["Замбия", "Лусака"],
        ["Зимбабве", "Хараре"]
      ]
    };
  },

  created() {
    this.activeUrl = this.ajaxApi[this.activeCard][1];
    for (let i = 0; i < 3; i++) {
      this.randomCountryIndex[i] = Math.round(Math.random() * 222);
      this.activeQuestCard[i] = this.ajaxApi[
        this.randomCountryIndex[i]
      ][1].toLowerCase();
      this.randomCountry[i] = this.ajaxApi[this.randomCountryIndex[i]][0];
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
      this.totalLetters++;
      this.$forceUpdate();
      this.correctInputLetters[index] = this.inputData[index];
      this.indexInputLetters[index] = this.indexInputLetters[index] + 1;
      this.validCheck(index);
    },

    setFocus: function(item) {
      const id = this.inputStatus.findIndex(elem => elem != "is-valid");
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
      this.letterCount = sessionStorage.setItem(
        "totalLetters",
        this.totalLetters
      );
      /*       if (sessionStorage.getItem("totalLetters") != null) {
        sessionStorage.setItem(
          "totalLetters",
          this.letterCount + this.totalLetters
        );
      } else sessionStorage.setItem("totalLetters", this.totalLetters);
      this.totalLetters; */
      this.nextStatusButton = true;
      location.reload();
    }
  }
};
</script>

<style scoped>
div {
  user-select: none;
}
#country {
  font-size: 35px;
}
#empty {
  color: white;
  font-size: 75px;
}
#count {
  margin: auto;
}
</style>
