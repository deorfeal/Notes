<script>
import { findProp } from '@vue/compiler-core'
import { nextTick } from 'vue'


export default {

  data() {
    return {
      tagItem: false,
      itemTagsArrow: 'menu-list__arrow--normal',
      isEptyListOfTexts: true,

      // Список ячеек текста - список textArea - активная ячейка соответсвенно 
      listOfTexts: [],
      listOfAreaTexts: [],
      allTextsLines: [],
      activeTextLine: 0,
      activeAreaTextObj: undefined,
      //

      // 
      itemsWithPinned: [],
      // 

      //
      allFavoritesList: [],
      allFavoriteCount: [],
      // 

      // 
      allNotes: true,
      allFavorites: false,
      // 

      // 
      deletePopup: false,
      // 

      // 
      inputValue: [],
      // 

    }
  },

  mounted() {
    if (this.listOfTexts.length == 0) {
      this.isEptyListOfTexts = true
    } else {
      this.isEptyListOfTexts = false
    };
  },

  watch: {

    // создаем углубленный watch который позволяет следить нам за изменениями объекта который внутри имеет нужное нам значение textarea - после чего присваиваем это значение активному элементу в списке там где название
    // при этом всем осуществляя проверку на длину нашего значения - и если оно больше чем надо, то добавляеться троеточие с определенного момента
    activeAreaTextObj: {
      handler() {
        if( window.innerWidth == 650 || window.innerWidth < 650 ) {
          if (this.listOfAreaTexts[this.activeTextLine].value.length > 15) {
            this.listOfTexts[this.activeTextLine].titleValue = this.listOfAreaTexts[this.activeTextLine].value.slice(0, 15) + '...'
          } else if (this.listOfAreaTexts[this.activeTextLine].value.length == 0) {
            this.listOfTexts[this.activeTextLine].titleValue = 'Your staff here'
          } else {
            this.listOfTexts[this.activeTextLine].titleValue = this.listOfAreaTexts[this.activeTextLine].value
          }
        } else {
          if (this.listOfAreaTexts[this.activeTextLine].value.length > 25) {
            this.listOfTexts[this.activeTextLine].titleValue = this.listOfAreaTexts[this.activeTextLine].value.slice(0, 25) + '...'
          } else if (this.listOfAreaTexts[this.activeTextLine].value.length == 0) {
            this.listOfTexts[this.activeTextLine].titleValue = 'Your staff here'
          } else {
            this.listOfTexts[this.activeTextLine].titleValue = this.listOfAreaTexts[this.activeTextLine].value
          }
        }
      },
      deep: true
    },
    // 

  },

  methods: {

    openTags() {
      if (this.tagItem == false) {
        this.tagItem = true
        this.itemTagsArrow = ''
      } else {
        this.tagItem = false
        this.itemTagsArrow = 'menu-list__arrow--normal'
      };
    },

    plusListItem() {

      // Добавляем при клике на плюсик общее количество элементов где все списки ( title ) activeTextLine и делаем последний элемент массива активным ( то есть его позицию - что в итоге являеться индексом, на чем потом все и строится)
      this.allTextsLines.push(0)
      this.activeTextLine = this.allTextsLines.length - 1
      // - 1 выше для того что бы получилось ноль - ведь везде нулевые индексы значит по дефолту после нажатия на плюс активный элемент нулевой ( то есть тот же первый )

      // Добавляет новый объект с значениями value и id в массив listOfAreaTexts - так же вставляет нужные айди элементам при помощи цикла for и позиций элементов в массиве  
      this.listOfAreaTexts.push({ value: '', id: 0 })
      for (let item of this.listOfAreaTexts) {
        item.id = this.listOfAreaTexts.indexOf(item)
      };
      // 

      //  Добавляет новый объект с значением titleValue и id в массив listOfTexts - так же вставляет нужные айди элементам при помощи цикла for и позиций элементов в массиве  
      this.listOfTexts.push({ titleValue: 'Your staff here', id: 0, });
      for (let item of this.listOfTexts) {
        item.id = this.listOfTexts.indexOf(item)
        item.order = this.listOfTexts.indexOf(item) + 1
        item.back = '' // Меняем стиль активности на ничего, когда добавляем новый элемент - всем остальным
      };


      this.listOfTexts[this.activeTextLine].back = '#f5f5f5' // Потом именно новому активному добавляем клас 
      // 

      // Добавляем нашему ключу в роли значения объект за изменениями которого будем следить ради того что бы работать с watch
      this.activeAreaTextObj = this.listOfAreaTexts[this.activeTextLine]
      // 

      // Скрываем или показывем блок где написано что ничего нет - в зависимости от значений которые меняються после наполнения массива с текстами ( где поле title )
      if (this.listOfTexts.length == 0) {
        this.isEptyListOfTexts = true
      } else {
        this.isEptyListOfTexts = false
      };
      // 


      // Делаем проверку находимся ли мы в поле favorite после чего делаем так что при добавлении элемента он сразу становится favorite и все уже связано с all notes
      if (this.allFavorites) {
        this.listOfTexts[this.activeTextLine].favorited = true

        // Добавляем новый элемент для счета длины и вывода количества в строку 
        let copyOfAllFavoriteCount = this.allFavoriteCount
        copyOfAllFavoriteCount.push(0)
        this.allFavoriteCount = copyOfAllFavoriteCount
        // 
      };
      //

      // Когда мы нажимаем на плюсик и добавляется элемент, то после этого в поле поиска появляется search вместо ( тут ничего нет )
      this.$refs.inputWithFindings.placeholder = 'Search...'
      // 

    },

    // Делаем активным тот елемент на который мы именно нажали ( перебивает плюсик ) - но плюсик перебивает когда мы нажимаем на него ( все видно в готовой работе при кликах )
    // Используеться currentTarget - что бы сразу получать элемент на котором стоит наш обработчик а не тот на который мы кликнули ( не на текст внутри допустим )
    addAcitveElement(el) {
      this.activeTextLine = +el.currentTarget.id

      // Добавляем нашему ключу в роли значения объект за изменениями которого будем следить ради того что бы работать с watch
      this.activeAreaTextObj = this.listOfAreaTexts[this.activeTextLine]
      // 

      // Добавляем отнимаем активный клас и меняем стиль
      for (let item of this.listOfTexts) {
        item.back = '' // Меняем стиль активности на ничего, когда добавляем новый элемент - всем остальным
      };

      this.listOfTexts[this.activeTextLine].back = '#f5f5f5' // Потом именно новому активному добавляем клас 
      // 
    },
    //

    appeareDeletePopup() {
      this.deletePopup = true
    },

    closeDeletePopup() {
      this.deletePopup = false
    },

    // Создаем аналоги наших массивов что бы потом вести над ними мутации, так как иначе реактивное состояние наших ключей не изменится - нашим ключам передаем новые значения с правильным количеством элементов после удаления 
    // Так же записываем новое значение для активной текстовой строки ( для активного элемента списка там где title )
    // Ведем проверку на то остался ли последний элемент в массиве - Если есть последний, чуть меняем код что бы очистить массивы 
    deleteTextItem() {
      if (this.listOfTexts.length == 1) {
        let listOfTextsArray = this.listOfTexts
        let listOfAreaTextsArray = this.listOfAreaTexts
        let allTextsLinesArray = this.allTextsLines

        listOfTextsArray.splice(this.activeTextLine, 1)
        listOfAreaTextsArray.splice(this.activeTextLine, 1)
        allTextsLinesArray.splice(this.activeTextLine, 1)

        this.listOfTexts = listOfTextsArray
        this.listOfAreaTexts = listOfAreaTextsArray
        this.allTextsLines = allTextsLinesArray

        this.activeTextLine = undefined
        this.isEptyListOfTexts = true
        this.deletePopup = false
      } else {

        this.deletePopup = false

        let listOfTextsArray = this.listOfTexts
        let listOfAreaTextsArray = this.listOfAreaTexts
        let allTextsLinesArray = this.allTextsLines

        listOfTextsArray.splice(this.activeTextLine, this.activeTextLine)
        listOfAreaTextsArray.splice(this.activeTextLine, this.activeTextLine)
        allTextsLinesArray.splice(this.activeTextLine, this.activeTextLine)

        this.listOfTexts = listOfTextsArray
        this.listOfAreaTexts = listOfAreaTextsArray
        this.allTextsLines = allTextsLinesArray

        this.activeTextLine = this.allTextsLines.length - 1

        // Добавляем нашему ключу в роли значения объект за изменениями которого будем следить ради того что бы работать с watch
        this.activeAreaTextObj = this.listOfAreaTexts[this.activeTextLine]
        // 

        // Добавляем отнимаем активный клас и меняем стиль
        for (let item of this.listOfTexts) {
          item.back = '' // Меняем стиль активности на ничего, когда добавляем новый элемент - всем остальным
        };

        this.listOfTexts[this.activeTextLine].back = '#f5f5f5' // Потом именно новому активному добавляем клас 
        // 
      }

      // Отнимаем элемент для счета длины и вывода количества в строку ( favorite )
      let copyOfAllFavoriteCount = this.allFavoriteCount
      copyOfAllFavoriteCount.splice(0, 1)
      this.allFavoriteCount = copyOfAllFavoriteCount
      // 
    },
    // 

    // Добавляем пины по клику на пин - делаем проверку для того что бы сделать как бы toggle и прочее махинации для того что бы список нормально менялся 
    addPin() {
      if (this.listOfTexts[this.activeTextLine].pinned == undefined || this.listOfTexts[this.activeTextLine].pinned == null || this.listOfTexts[this.activeTextLine].pinned == false) {
        this.listOfTexts[this.activeTextLine].pinned = true

        for (let item of this.listOfTexts) {
          item.order++
        }

        this.listOfTexts[this.activeTextLine].order = 1

        let ourItemsWithPinned = this.itemsWithPinned
        ourItemsWithPinned.push(0)
        this.itemsWithPinned = ourItemsWithPinned

      } else {
        this.listOfTexts[this.activeTextLine].pinned = false

        let ourItemsWithPinned = this.itemsWithPinned
        ourItemsWithPinned.splice(0, 0)
        this.itemsWithPinned = ourItemsWithPinned

        let digitsOfOrder = []

        for (let item of this.listOfTexts) {
          if (item.pinned) {
            if (this.itemsWithPinned.length === this.allTextsLines.length) {
              digitsOfOrder.push(item.order)
              this.listOfTexts[this.activeTextLine].order = Math.max(...digitsOfOrder) + 1
              if (this.itemsWithPinned.length === 1) {
                this.listOfTexts[this.activeTextLine].order = Math.max(...digitsOfOrder) + 1
              }
            }
            else {
              this.listOfTexts[this.activeTextLine].order = this.listOfTexts[this.activeTextLine].id
            }
          }
        }

      }
    },
    // 

    addFavorite() {
      if (this.listOfTexts[this.activeTextLine].favorited == undefined || this.listOfTexts[this.activeTextLine].favorited == null || this.listOfTexts[this.activeTextLine].favorited == false) {
        this.listOfTexts[this.activeTextLine].favorited = true

        // Добавляем новый элемент для счета длины и вывода количества в строку 
        let copyOfAllFavoriteCount = this.allFavoriteCount
        copyOfAllFavoriteCount.push(0)
        this.allFavoriteCount = copyOfAllFavoriteCount
        // 
      } else {
        this.listOfTexts[this.activeTextLine].favorited = false

        // Отнимаем элемент для счета длины и вывода количества в строку 
        let copyOfAllFavoriteCount = this.allFavoriteCount
        copyOfAllFavoriteCount.splice(0, 1)
        this.allFavoriteCount = copyOfAllFavoriteCount
        // 
      }
    },

    favoriteChange() {
      this.allNotes = false
      this.allFavorites = true
      // Создаем проверку для того что бы если не будет фейворит запесей - то вывести текст о том что ничего нет
      if (this.allFavoriteCount.length == 0) {
        this.isEptyListOfTexts = true
      }
      //
    },

    allNoteChange() {
      this.allNotes = true
      this.allFavorites = false
      if (this.allTextsLines.length != 0) {
        this.isEptyListOfTexts = false
      }
    },

    searchItem(event) {
      let seachInputValue = event.currentTarget.value
      let arrayOfInputSymbols = []

      for (let symbolOfInput of seachInputValue) {
        arrayOfInputSymbols.push(symbolOfInput)
      }

      this.inputValue = arrayOfInputSymbols

      for (let ourItem of this.listOfTexts) {

        let arrOfSymbols = []

        for (let symbol of ourItem.titleValue) {
          arrOfSymbols.push(symbol)
        }

        for (let symbolOfValue of this.inputValue) {
          for (let symbolOfOurArr of arrOfSymbols) {
            if (symbolOfValue == symbolOfOurArr) {
              let ourLiElements = document.querySelectorAll('.main-search-list__item')
              for (let elem of ourLiElements) {
                if (elem.id == ourItem.id) {
                  elem.classList.add('fined')
                } else {
                  elem.classList.remove('fined')
                }
              }
            }
            if (this.inputValue.length == 1) {
              let ourLiElements = document.querySelectorAll('.main-search-list__item')
              for (let elem of ourLiElements) {
                elem.classList.remove('fined')
              }
            }
          }
        }

      }


    },
    menuToActive() {
      if ( this.$refs.menuInner.classList.contains('menu__inner--active') ) {
        this.$refs.menuInner.classList.remove('menu__inner--active')
      } else {
        this.$refs.menuInner.classList.add('menu__inner--active')
      }
    }
  },

}

</script>

<!-- <script>

  export default {
    name: "ім'я нашого компонента"
  }

</script> -->

<template>
  <main class="main">
    <div class="main__menu menu">
      <div ref="menuInner" class="menu__inner">
        <p class="menu__text">
          Menu
        </p>
        <ul class="menu__list menu-list">
          <li @click="allNoteChange" class="menu-list__item" :class="{'menu-list__item--active': allNotes}">
            <div class="menu-list__box">
              <svg class="menu-list__box-svg menu-list__box-svg--note" height="800px" width="800px" version="1.1"
                id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 512 512" xml:space="preserve">
                <g>
                  <path style="fill:#EDEBDA;" d="M368.814,503.322H4.339V0h286.373l34.712,34.712l43.39,43.39V503.322z" />
                  <g>
                    <path style="fill:#CEC9AE;" d="M290.712,78.102h78.102L334.102,43.39L290.712,0V78.102z" />
                    <path style="fill:#CEC9AE;" d="M290.712,234.305H82.441c-4.79,0-8.678-3.888-8.678-8.678s3.888-8.678,8.678-8.678h208.271
        c4.79,0,8.678,3.888,8.678,8.678S295.502,234.305,290.712,234.305" />
                    <path style="fill:#CEC9AE;" d="M256,286.373H117.153c-4.79,0-8.678-3.888-8.678-8.678c0-4.79,3.888-8.678,8.678-8.678H256
        c4.79,0,8.678,3.888,8.678,8.678C264.678,282.485,260.79,286.373,256,286.373" />
                    <path style="fill:#CEC9AE;" d="M290.712,338.441H82.441c-4.79,0-8.678-3.888-8.678-8.678c0-4.79,3.888-8.678,8.678-8.678h208.271
        c4.79,0,8.678,3.888,8.678,8.678C299.39,334.553,295.502,338.441,290.712,338.441" />
                    <path style="fill:#CEC9AE;" d="M256,390.508H117.153c-4.79,0-8.678-3.888-8.678-8.678c0-4.79,3.888-8.678,8.678-8.678H256
        c4.79,0,8.678,3.888,8.678,8.678C264.678,386.621,260.79,390.508,256,390.508" />
                    <path style="fill:#CEC9AE;" d="M256,182.237H117.153c-4.79,0-8.678-3.888-8.678-8.678s3.888-8.678,8.678-8.678H256
        c4.79,0,8.678,3.888,8.678,8.678S260.79,182.237,256,182.237" />
                  </g>
                </g>
              </svg>
              <p class="menu-list__box-text">
                All Notes
              </p>
            </div>
            <p class="menu-list__count">
              {{ allTextsLines.length }}
            </p>
          </li>
          <li @click="favoriteChange" class="menu-list__item" :class="{ 'menu-list__item--active': allFavorites }">
            <div class="menu-list__box">
              <svg class="menu-list__box-svg menu-list__box-svg--favorite" version="1.1" id="Layer_1"
                xmlns:x="&ns_extend;" xmlns:i="&ns_ai;" xmlns:graph="&ns_graphs;" xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px" viewBox="0 0 750 750"
                style="enable-background:new 0 0 750 750;" xml:space="preserve">
                <switch>
                  <foreignObject requiredExtensions="&ns_ai;" x="0" y="0" width="1" height="1">
                    <i:pgfRef xlink:href="#adobe_illustrator_pgf">
                    </i:pgfRef>
                  </foreignObject>
                  <g i:extraneous="self">
                    <path
                      d="M504.6,445.4l80.1,232.4L376.6,534.5L170.4,677.8l78.5-231.3l-204.8-141h252.6L376,72.2l80.4,233.3h249.5L504.6,445.4z" />
                  </g>
                </switch>
              </svg>
              <p class="menu-list__box-text">
                Favorite
              </p>
            </div>
            <p class="menu-list__count">
              {{ allFavoriteCount.length }}
            </p>
          </li>
          <!-- <li @click="openTags()" class="menu-list__item menu-list__item--tags">
          <div class="menu-list__box">
            <svg class="menu-list__box-svg menu-list__box-svg--tag" fill="#000000" height="800px" width="800px"
              version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
              viewBox="0 0 512.001 512.001" xml:space="preserve">
              <g>
                <g>
                  <path d="M506.513,311.066L253.87,58.423c-2.024-2.026-4.486-3.559-7.195-4.483L140.15,17.593
        c-6.758-2.305-14.242-0.568-19.294,4.483L84.709,58.222L31.977,5.491c-7.314-7.315-19.176-7.315-26.49,0
        c-7.315,7.315-7.315,19.175,0,26.49l52.732,52.731l-36.14,36.141c-5.051,5.05-6.79,12.534-4.483,19.294L53.943,246.67
        c0.924,2.71,2.458,5.172,4.483,7.197L311.071,506.51c7.314,7.315,19.175,7.315,26.49,0l168.954-168.954
        C513.83,330.241,513.83,318.382,506.513,311.066z M227.241,227.238c-21.817,21.819-57.132,21.82-78.952,0
        c-21.768-21.768-21.768-57.185,0.001-78.953c21.817-21.819,57.132-21.82,78.953,0C249.009,170.053,249.009,205.47,227.241,227.238
        z" />
                </g>
              </g>
            </svg>
            <p class="menu-list__box-text">
              Tags
            </p>
          </div>
          <button @click="openTags" class="menu-list__arrow" type="button">
            <svg :class="itemTagsArrow" width="32" height="17" viewBox="0 0 32 17" fill="none"
              xmlns="http://www.w3.org/2000/svg">
              <path
                d="M30.8215 0.479573C30.5879 0.251242 30.2744 0.123413 29.948 0.123413C29.6215 0.123413 29.308 0.251242 29.0745 0.479573L16.0005 13.2361L2.92922 0.479573C2.69568 0.251242 2.38217 0.123413 2.05571 0.123413C1.72925 0.123413 1.41573 0.251242 1.18219 0.479573C1.06831 0.590557 0.977801 0.723252 0.915997 0.869826C0.854194 1.0164 0.822354 1.17388 0.822354 1.33297C0.822354 1.49207 0.854194 1.64955 0.915997 1.79612C0.977801 1.9427 1.06831 2.07539 1.18219 2.18638L15.0878 15.7518C15.332 15.9901 15.6595 16.1234 16.0005 16.1234C16.3415 16.1234 16.6691 15.9901 16.9133 15.7518L30.8189 2.18638C30.9327 2.07539 31.0233 1.9427 31.0851 1.79612C31.1469 1.64955 31.1787 1.49207 31.1787 1.33297C31.1787 1.17388 31.1469 1.0164 31.0851 0.869826C31.0233 0.723252 30.9327 0.590557 30.8189 0.479573H30.8215Z"
                fill="black" />
            </svg>
          </button>
        </li>
        <li v-if="tagItem" class="menu-list__item menu-list__item--tag">
          <div class="menu-list__box">
            <p class="menu-list__box-text">
              Tag
            </p>
          </div>
          <p class="menu-list__count">0</p>
        </li> -->
        </ul>
        <a class="menu__link" href="https://github.com/deorfeal">
          Github
        </a>
      </div>
      <button @click="menuToActive" class="menu__btn" type="button">
        <span>

        </span>
      </button>
    </div>
    <div class="main__search main-search">
      <div class="main-search__top main-search-top">
        <form class="main-search-top__form" action="#">
          <input ref="inputWithFindings" @input="searchItem" class="main-search-top__form-input"
            placeholder="Is nothing here" type="search">
          <button class="main-search-top__form-button" type="button">
            <svg width="27" height="32" viewBox="0 0 27 32" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd" clip-rule="evenodd"
                d="M21.9996 11.6472C21.9996 16.9753 17.6807 21.2943 12.3536 21.2943C7.02638 21.2943 2.70752 16.9753 2.70752 11.6472C2.70752 6.319 7.02638 2 12.3536 2C17.6807 2 21.9996 6.319 21.9996 11.6472ZM17.8076 21.9407C16.1806 22.8048 14.3242 23.2943 12.3536 23.2943C5.92163 23.2943 0.70752 18.0797 0.70752 11.6472C0.70752 5.21461 5.92163 0 12.3536 0C18.7855 0 23.9996 5.21461 23.9996 11.6472C23.9996 15.3899 22.2344 18.7203 19.4912 20.8511L26.5258 30.1465L24.931 31.3534L17.8076 21.9407Z"
                fill="white" fill-opacity="0.2" />
            </svg>
          </button>
        </form>
        <button @click="plusListItem()" class="main-search-top__add-button" type="button">
          +
        </button>
      </div>
      <div class="main-search__row">
        <p class="main-search__row-text">
          Title
        </p>
        <button class="main-search__row-btn">
          <svg width="32" height="17" viewBox="0 0 32 17" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path
              d="M30.8215 0.479573C30.5879 0.251242 30.2744 0.123413 29.948 0.123413C29.6215 0.123413 29.308 0.251242 29.0745 0.479573L16.0005 13.2361L2.92922 0.479573C2.69568 0.251242 2.38217 0.123413 2.05571 0.123413C1.72925 0.123413 1.41573 0.251242 1.18219 0.479573C1.06831 0.590557 0.977801 0.723252 0.915997 0.869826C0.854194 1.0164 0.822354 1.17388 0.822354 1.33297C0.822354 1.49207 0.854194 1.64955 0.915997 1.79612C0.977801 1.9427 1.06831 2.07539 1.18219 2.18638L15.0878 15.7518C15.332 15.9901 15.6595 16.1234 16.0005 16.1234C16.3415 16.1234 16.6691 15.9901 16.9133 15.7518L30.8189 2.18638C30.9327 2.07539 31.0233 1.9427 31.0851 1.79612C31.1469 1.64955 31.1787 1.49207 31.1787 1.33297C31.1787 1.17388 31.1469 1.0164 31.0851 0.869826C31.0233 0.723252 30.9327 0.590557 30.8189 0.479573H30.8215Z"
              fill="black" />
          </svg>
        </button>
      </div>
      <TransitionGroup>
        <ul v-if="allNotes" class="main-search__list main-search-list">
          <TransitionGroup>
            <li v-if="(isEptyListOfTexts)" class="main-search-list__item">
              <p class="main-search-list__text main-search-list__text--empty">
                Is nothing here
              </p>
            </li>
          </TransitionGroup>
          <!-- Создаем узлы при помощи v-for наблюдая за количеством элементов в массиве - привязываем ключ и айдишник, так же при помощи информации из массива и отображаем название при помощи значения в массиве -->
          <TransitionGroup>
            <li @click="addAcitveElement" v-for="item in listOfTexts"
              :style="{ order: item.order, background: item.back }" :key="item.id" :id="item.id"
              class="main-search-list__item">
              <p class="main-search-list__text">
                {{ item.titleValue }}
              </p>
              <div class="main-search-list__box">
                <Transition>
                  <button v-if="item.favorited" :id="item.id" class="main-search-list__favorite">
                    <svg version="1.1" id="Layer_1" xmlns:x="&ns_extend;" xmlns:i="&ns_ai;" xmlns:graph="&ns_graphs;"
                      xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
                      viewBox="0 0 750 750" style="enable-background:new 0 0 750 750;" xml:space="preserve">
                      <switch>
                        <foreignObject requiredExtensions="&ns_ai;" x="0" y="0" width="13" height="13">
                          <i:pgfRef xlink:href="#adobe_illustrator_pgf">
                          </i:pgfRef>
                        </foreignObject>
                        <g i:extraneous="self">
                          <path
                            d="M504.6,445.4l80.1,232.4L376.6,534.5L170.4,677.8l78.5-231.3l-204.8-141h252.6L376,72.2l80.4,233.3h249.5L504.6,445.4z" />
                        </g>
                      </switch>
                    </svg>
                  </button>
                </Transition>
                <Transition>
                  <button v-if="item.pinned" :id="item.id" class="main-search-list__pin">
                    <svg height="13px" width="13px" version="1.1" id="_x32_" xmlns="http://www.w3.org/2000/svg"
                      xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 512 512" xml:space="preserve">
                      <g>
                        <polygon class="st0"
                          points="85.564,392.665 23.785,454.449 0,508.121 53.677,484.337 180.196,357.818 150.304,327.92 	" />
                        <path class="st0" d="M422.416,93.462L332.837,3.879c-31.304,31.308-12.697,75.686-12.697,75.686l-132.209,84.362
		c-43.25-22.714-103.589-5.941-133.414,23.884L328.051,461.36c29.843-29.824,46.625-90.166,23.902-133.422l84.351-132.199
		c0,0,44.388,18.606,75.696-12.693L422.416,93.462z" />
                      </g>
                    </svg>
                  </button>
                </Transition>
              </div>
            </li>
          </TransitionGroup>
          <!--  -->
        </ul>
      </TransitionGroup>
      <TransitionGroup>
        <ul v-if="allFavorites" class="main-search__list main-search-list">
          <TransitionGroup>
            <li v-if="(isEptyListOfTexts)" class="main-search-list__item">
              <p class="main-search-list__text main-search-list__text--empty">
                Is nothing here
              </p>
            </li>
          </TransitionGroup>
          <!-- Создаем узлы при помощи v-for наблюдая за количеством элементов в массиве - привязываем ключ и айдишник, так же при помощи информации из массива и отображаем название при помощи значения в массиве -->

          <template v-for="item in listOfTexts">
            <li @click="addAcitveElement" v-if="item.favorited" :style="{ order: item.order, background: item.back }"
              :key="item.id" :id="item.id" class="main-search-list__item">
              <p class="main-search-list__text">
                {{ item.titleValue }}
              </p>
              <div class="main-search-list__box">
                <Transition>
                  <button v-if="item.favorited" :id="item.id" class="main-search-list__favorite">
                    <svg version="1.1" id="Layer_1" xmlns:x="&ns_extend;" xmlns:i="&ns_ai;" xmlns:graph="&ns_graphs;"
                      xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
                      viewBox="0 0 750 750" style="enable-background:new 0 0 750 750;" xml:space="preserve">
                      <switch>
                        <foreignObject requiredExtensions="&ns_ai;" x="0" y="0" width="13" height="13">
                          <i:pgfRef xlink:href="#adobe_illustrator_pgf">
                          </i:pgfRef>
                        </foreignObject>
                        <g i:extraneous="self">
                          <path
                            d="M504.6,445.4l80.1,232.4L376.6,534.5L170.4,677.8l78.5-231.3l-204.8-141h252.6L376,72.2l80.4,233.3h249.5L504.6,445.4z" />
                        </g>
                      </switch>
                    </svg>
                  </button>
                </Transition>
                <Transition>
                  <button v-if="item.pinned" :id="item.id" class="main-search-list__pin">
                    <svg height="13px" width="13px" version="1.1" id="_x32_" xmlns="http://www.w3.org/2000/svg"
                      xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 512 512" xml:space="preserve">
                      <g>
                        <polygon class="st0"
                          points="85.564,392.665 23.785,454.449 0,508.121 53.677,484.337 180.196,357.818 150.304,327.92 	" />
                        <path class="st0" d="M422.416,93.462L332.837,3.879c-31.304,31.308-12.697,75.686-12.697,75.686l-132.209,84.362
		c-43.25-22.714-103.589-5.941-133.414,23.884L328.051,461.36c29.843-29.824,46.625-90.166,23.902-133.422l84.351-132.199
		c0,0,44.388,18.606,75.696-12.693L422.416,93.462z" />
                      </g>
                    </svg>
                  </button>
                </Transition>
              </div>
            </li>
          </template>

          <!--  -->
        </ul>
      </TransitionGroup>
      <!-- <button class="main-search-list__btn">
        <svg version="1.1" id="Layer_1" xmlns:x="&ns_extend;" xmlns:i="&ns_ai;" xmlns:graph="&ns_graphs;"
          xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
          viewBox="0 0 750 750" style="enable-background:new 0 0 750 750;" xml:space="preserve">
          <switch>
            <foreignObject requiredExtensions="&ns_ai;" x="0" y="0" width="1" height="1">
              <i:pgfRef xlink:href="#adobe_illustrator_pgf">
              </i:pgfRef>
            </foreignObject>
            <g i:extraneous="self">
              <path
                d="M504.6,445.4l80.1,232.4L376.6,534.5L170.4,677.8l78.5-231.3l-204.8-141h252.6L376,72.2l80.4,233.3h249.5L504.6,445.4z" />
            </g>
          </switch>
        </svg>
      </button>  -->
    </div>
    <div class="main__markdown main-markdown">
      <div class="main-markdown__top main-markdown-top">
        <div class="main-markdown-top__wrapper">
          <!-- <div class="main-markdown-top__row">
            <button class="main-markdown-top__row-btn">
              <svg fill="#000000" height="13px" width="13px" version="1.1" id="Capa_1"
                xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 306.637 306.637" xml:space="preserve">
                <g>
                  <g>
                    <path d="M12.809,238.52L0,306.637l68.118-12.809l184.277-184.277l-55.309-55.309L12.809,238.52z M60.79,279.943l-41.992,7.896
			l7.896-41.992L197.086,75.455l34.096,34.096L60.79,279.943z" />
                    <path d="M251.329,0l-41.507,41.507l55.308,55.308l41.507-41.507L251.329,0z M231.035,41.507l20.294-20.294l34.095,34.095
			L265.13,75.602L231.035,41.507z" />
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                  <g>
                  </g>
                </g>
              </svg>
            </button>
            <button class="main-markdown-top__row-btn">
              <svg fill="#000000" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink" width="13px" height="13px" viewBox="0 0 950 950"
                xml:space="preserve">
                <g>
                  <path d="M857.7,141.3c-30.1-30.1-65.1-53.5-104.3-69.4c-37.8-15.3-77.7-23.2-118.7-23.2c-40.9,0-80.9,7.7-118.7,22.9
               c-39.1,15.8-74.2,38.9-104.3,68.8L73.1,478.3C49.3,501.9,30.9,529.4,18.3,560.2C6.2,589.9,0,621.3,0,653.6
               C0,685.7,6.1,717,18.1,746.7c12.4,30.7,30.7,58.2,54.3,81.899c23.6,23.7,51.2,42,81.9,54.5c29.7,12.101,61.1,18.2,93.3,18.2
               c32.2,0,63.6-6.1,93.3-18.1c30.8-12.5,58.399-30.8,82.1-54.4l269.101-268c17.3-17.2,30.6-37.3,39.699-59.7
               c8.801-21.6,13.2-44.5,13.2-67.899c0-48.2-18.8-93.2-52.899-127c-34-34.2-79.2-53.1-127.301-53.3c-48.199-0.1-93.5,18.6-127.6,52.7
               L269.6,473.3c-8.5,8.5-13.1,19.7-13.1,31.601c0,11.899,4.6,23.199,13.1,31.6l0.7,0.7c17.4,17.5,45.8,17.5,63.3,0.1l168-167.5
               c35.1-34.8,92.1-35,127.199-0.399c16.9,16.8,26.101,39.3,26.101,63.399c0,24.3-9.4,47.101-26.5,64.101l-269,268
               c-0.5,0.5-0.9,0.899-1.2,1.5c-29.7,28.899-68.9,44.699-110.5,44.5c-41.9-0.2-81.2-16.5-110.6-46c-14.7-15-26.1-32.5-34-52
               C95.5,694,91.7,674,91.7,653.6c0-41.8,16.1-80.899,45.4-110.3c0.4-0.3,0.7-0.6,1.1-0.899l337.9-337.8c0.3-0.3,0.6-0.7,0.899-1.1
               c21.4-21,46.3-37.4,74-48.5c27-10.8,55.4-16.2,84.601-16.2c29.199,0,57.699,5.6,84.6,16.4c27.9,11.3,52.9,27.8,74.3,49.1
               c21.4,21.4,37.9,46.4,49.2,74.3c10.9,26.9,16.4,55.4,16.4,84.6c0,29.3-5.5,57.9-16.5,85c-11.301,28-28,53.2-49.5,74.8l-233.5,232.8
               c-8.5,8.5-13.2,19.7-13.2,31.7s4.7,23.2,13.1,31.6l0.5,0.5c17.4,17.4,45.8,17.4,63.2,0L857.5,586.9
               C887.601,556.8,911,521.7,926.9,482.6C942.3,444.8,950,404.9,950,363.9c0-40.9-7.8-80.8-23.1-118.5
               C911.101,206.3,887.8,171.3,857.7,141.3z" />
                </g>
              </svg>
            </button> 
            <button class="main-markdown-top__row-btn">
              <svg fill="#000000" height="13px" width="13px" version="1.1" id="Layer_1"
                xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
                viewBox="0 0 512.001 512.001" xml:space="preserve">
                <g>
                  <g>
                    <path d="M506.513,311.066L253.87,58.423c-2.024-2.026-4.486-3.559-7.195-4.483L140.15,17.593
                 c-6.758-2.305-14.242-0.568-19.294,4.483L84.709,58.222L31.977,5.491c-7.314-7.315-19.176-7.315-26.49,0
                 c-7.315,7.315-7.315,19.175,0,26.49l52.732,52.731l-36.14,36.141c-5.051,5.05-6.79,12.534-4.483,19.294L53.943,246.67
                 c0.924,2.71,2.458,5.172,4.483,7.197L311.071,506.51c7.314,7.315,19.175,7.315,26.49,0l168.954-168.954
                 C513.83,330.241,513.83,318.382,506.513,311.066z M227.241,227.238c-21.817,21.819-57.132,21.82-78.952,0
                 c-21.768-21.768-21.768-57.185,0.001-78.953c21.817-21.819,57.132-21.82,78.953,0C249.009,170.053,249.009,205.47,227.241,227.238
                 z" />
                  </g>
                </g>
              </svg>
            </button>
          </div> -->
          <div class="main-markdown-top__row">
            <button @click="addFavorite" class="main-markdown-top__row-btn">
              <svg width="13px" height="13px" version="1.1" id="Layer_1" xmlns:x="&ns_extend;" xmlns:i="&ns_ai;"
                xmlns:graph="&ns_graphs;" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
                x="0px" y="0px" viewBox="0 0 750 750" style="enable-background:new 0 0 750 750;" xml:space="preserve">
                <switch>
                  <foreignObject requiredExtensions="&ns_ai;" x="0" y="0" width="1" height="1">
                    <i:pgfRef xlink:href="#adobe_illustrator_pgf">
                    </i:pgfRef>
                  </foreignObject>
                  <g i:extraneous="self">
                    <path
                      d="M504.6,445.4l80.1,232.4L376.6,534.5L170.4,677.8l78.5-231.3l-204.8-141h252.6L376,72.2l80.4,233.3h249.5L504.6,445.4z" />
                  </g>
                </switch>
              </svg>
            </button>
            <button @click="addPin" class="main-markdown-top__row-btn">
              <svg height="13px" width="13px" version="1.1" id="_x32_" xmlns="http://www.w3.org/2000/svg"
                xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 512 512" xml:space="preserve">
                <g>
                  <polygon class="st0"
                    points="85.564,392.665 23.785,454.449 0,508.121 53.677,484.337 180.196,357.818 150.304,327.92 	" />
                  <path class="st0" d="M422.416,93.462L332.837,3.879c-31.304,31.308-12.697,75.686-12.697,75.686l-132.209,84.362
		c-43.25-22.714-103.589-5.941-133.414,23.884L328.051,461.36c29.843-29.824,46.625-90.166,23.902-133.422l84.351-132.199
		c0,0,44.388,18.606,75.696-12.693L422.416,93.462z" />
                </g>
              </svg>
            </button>
          </div>
        </div>
        <div class="main-markdown-top__row">
          <button @click="(appeareDeletePopup)" class="main-markdown-top__row-btn">
            <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"
              width="13px" height="13px" viewBox="0 0 256 256" xml:space="preserve">
              <defs>
              </defs>
              <g style="stroke: none; stroke-width: 0; stroke-dasharray: none; stroke-linecap: butt; stroke-linejoin: miter; stroke-miterlimit: 10; fill: none; fill-rule: nonzero; opacity: 1;"
                transform="translate(1.4065934065934016 1.4065934065934016) scale(2.81 2.81)">
                <path
                  d="M 76.777 2.881 H 57.333 V 2.412 C 57.333 1.08 56.253 0 54.921 0 H 35.079 c -1.332 0 -2.412 1.08 -2.412 2.412 v 0.469 H 13.223 c -1.332 0 -2.412 1.08 -2.412 2.412 v 9.526 c 0 1.332 1.08 2.412 2.412 2.412 h 63.554 c 1.332 0 2.412 -1.08 2.412 -2.412 V 5.293 C 79.189 3.961 78.109 2.881 76.777 2.881 z"
                  style="stroke: none; stroke-width: 1; stroke-dasharray: none; stroke-linecap: butt; stroke-linejoin: miter; stroke-miterlimit: 10; fill: rgb(0,0,0); fill-rule: nonzero; opacity: 1;"
                  transform=" matrix(1 0 0 1 0 0) " stroke-linecap="round" />
                <path
                  d="M 73.153 22.119 H 16.847 c -1.332 0 -2.412 1.08 -2.412 2.412 v 63.057 c 0 1.332 1.08 2.412 2.412 2.412 h 56.306 c 1.332 0 2.412 -1.08 2.412 -2.412 V 24.531 C 75.565 23.199 74.485 22.119 73.153 22.119 z M 33.543 81.32 c 0 1.332 -1.08 2.412 -2.412 2.412 h -2.245 c -1.332 0 -2.412 -1.08 -2.412 -2.412 V 30.799 c 0 -1.332 1.08 -2.412 2.412 -2.412 h 2.245 c 1.332 0 2.412 1.08 2.412 2.412 V 81.32 z M 48.535 81.32 c 0 1.332 -1.08 2.412 -2.412 2.412 h -2.245 c -1.332 0 -2.412 -1.08 -2.412 -2.412 V 30.799 c 0 -1.332 1.08 -2.412 2.412 -2.412 h 2.245 c 1.332 0 2.412 1.08 2.412 2.412 V 81.32 z M 63.526 81.32 c 0 1.332 -1.08 2.412 -2.412 2.412 h -2.245 c -1.332 0 -2.412 -1.08 -2.412 -2.412 V 30.799 c 0 -1.332 1.08 -2.412 2.412 -2.412 h 2.245 c 1.332 0 2.412 1.08 2.412 2.412 V 81.32 z"
                  style="stroke: none; stroke-width: 1; stroke-dasharray: none; stroke-linecap: butt; stroke-linejoin: miter; stroke-miterlimit: 10; fill: rgb(0,0,0); fill-rule: nonzero; opacity: 1;"
                  transform=" matrix(1 0 0 1 0 0) " stroke-linecap="round" />
              </g>
            </svg>
          </button>
        </div>
      </div>
      <div class="main-markdown__text">
        <!-- Добавляет новые элементы textarea после клика из массива при помощи v-for, так же привязивает к элементу айди и ключ с массива и делает двойное связывние значения value  -->
        <!-- Ми добавляємо template для того щоб ми могли використовувати v-if з item - і показуємо наш textarea лише якщо його айді співпадає з активним айді вибраного пункту текстів -->
        <template v-for="item in listOfAreaTexts">
          <!-- Тут мы делаем проверку на то пустной ли наш лист с текстами для того что бы отображать или нет техтарею в favorite -->
          <textarea tabindex="1" ref="textttarea" :key="item.id" :id="item.id" v-model="item.value"
            v-if="isEptyListOfTexts == false && item.id == this.activeTextLine" placeholder="Your staff here"
            class="main-markdown__text-textarea">
          </textarea>
        </template>
        <!--  -->
      </div>
    </div>
    <Teleport to="body">
      <Transition>
        <div v-if="deletePopup" class="deletePopup">
          <div class="deletePopup__box deletePopup-box">
            <p class="deletePopup-box__text">
              Warning !
            </p>
            <p class="deletePopup-box__subtext">
              You are about to delete this note, are you sure ?
            </p>
            <div class="deletePopup-box__row">
              <button @click="closeDeletePopup" class="deletePopup-box__row-cancel" type="button">
                Cancel
              </button>
              <button @click="deleteTextItem" class="deletePopup-box__row-sure" type="button">
                I'm sure
              </button>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>

  </main>
</template>

<style lang="scss">
@import './assets/style.scss';

#app {
  min-height: 100%;
  display: flex;
}

.main {
  display: flex;
  flex-grow: 1;
  min-height: 100%;
}

.menu {
  min-width: 300px;
  background: #20262a;
  &__btn {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 30px;
    height: 20px;
    display: none;
    position: relative;
    &::before {
      content: '';
      width: 100%;
      height: 2px;
      background: #fff;
      position: absolute;
      top: 0;
    }
    &::after {
      content: '';
      width: 100%;
      height: 2px;
      background: #fff;
      position: absolute;
      bottom: 0;
    }
    span {
      background: #fff;
      width: 100%;
      height: 2px;
    }
  }
  &__inner {
    min-height: 100%;
    flex-direction: column;
    display: flex;
  }

  &__text {
    font-size: 18px;
    color: #757575;
    padding: 10px 10px 10px 10px;
  }

  &__link {
    color: #f5f5f5;
    font-size: 18px;
    margin: 10px;
  }

}

.menu-list {
  flex-grow: 1;

  &__item {
    padding: 10px 10px 10px 20px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    cursor: pointer;
    transition: background 0.3s;

    &:hover {
      background-color: rgba(0, 0, 0, .188);
    }
  }

  &__item--active {
    background: #141718;
  }

  &__arrow {
    svg {
      transition: transform 0.3s;
    }
  }

  &__arrow--normal {
    transform: rotate(90deg);
    transition: transform 0.3s;
  }

  &__item--tag {
    display: flex;
    justify-content: space-between;

    .menu-list__box {
      margin-left: 30px;
    }
  }

  &__item-tag {
    display: flex;
    flex-direction: column;
    min-width: 100%;
    align-items: flex-start;
  }

  &__box {
    display: flex;
    gap: 10px;
    justify-content: center;
    align-items: center;
  }

  &__box-svg {
    display: flex;
    justify-content: center;
    align-items: center;
    fill: #fff;
  }

  &__box-svg--note {
    width: 20px;
    height: 20px;

    g {
      path {
        fill: #fff !important;
      }

      g {
        path:nth-child(1) {
          fill: transparent !important;
        }

        path {
          fill: #20262a !important;
        }
      }
    }
  }

  &__box-svg--favorite {
    width: 20px;
    height: 20px;
  }

  &__box-svg--tag {
    width: 20px;
    height: 20px;
  }

  &__box-text {
    color: #f5f5f5;
    font-size: 14px;
  }

  &__count {
    margin-right: 10px;
    color: #f5f5f5;
  }

  &__arrow {
    width: 29px;
    height: 25px;
    display: flex;
    justify-content: center;
    align-items: center;

    svg {
      width: 10px;
      height: 10px;

      path {
        stroke: #fff;
        fill: #f5f5f5 !important;
      }
    }
  }
}

// // // // // //  


.main-search {
  min-width: 300px;
  border: 2px solid #e0e0e0;
  border-top: unset;

  &__row {
    cursor: pointer;
    border: 2px solid #e0e0e0;
    border-left: unset;
    border-top: unset;
    border-right: unset;
    padding: 10px 20px 10px 10px;
    display: flex;
    justify-content: space-between;
  }

  &__row-text {
    font-size: 16px;
    color: #757575;
  }

  &__row-btn {
    svg {
      width: 10px;
      height: 10px;

      path {
        fill: #757575;
      }

    }
  }
}

.main-search-top {
  display: flex;
  align-items: center;
  background: #f5f5f5;
  padding: 10px 10px;
  gap: 10px;
  border-bottom: 2px solid #e0e0e0;

  &__form {
    width: 100%;
    display: flex;
    background: #fff;
    padding-right: 10px;
    align-items: center;
    height: 28px;
    gap: 15px;
    justify-content: space-between;
    border: 1px solid #e0e0e0;
    border-radius: 5px;

    svg {
      width: 15px;
      height: 15px;

      path {
        fill: #000;
        fill-opacity: 1;
      }
    }
  }

  &__form-input {
    color: #20262a;
    padding: 5px 0 5px 10px;
    background: unset;
    color: #000;
    font-size: 13px;
    width: 100%;

    &[type=text]::-ms-clear {
      display: none;
      width: 0;
      height: 0;
    }

    &[type=text]::-ms-reveal {
      display: none;
      width: 0;
      height: 0;
    }

    &[type="search"]::-webkit-search-decoration,
    &[type="search"]::-webkit-search-cancel-button,
    &[type="search"]::-webkit-search-results-button,
    &[type="search"]::-webkit-search-results-decoration {
      display: none;
    }

    &::placeholder {
      color: #757575;
      font-size: 13px;
    }
  }

  &__form-button {
    width: 15px;
    height: 15px;
  }

  &__add-button {
    width: 36px;
    height: 28px;
    background: #fff;
    color: #000;
    font-size: 16px;
    border: 1px solid #e0e0e0;
    border-radius: 5px;
    transition: background 0.3s;

    &:hover {
      background: #e0e0e0;
    }
  }
}

.main-search-list {
  display: flex;
  flex-direction: column;

  &__box {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 5px;
  }

  &__item {
    cursor: pointer;
    padding: 10px 17px 10px 10px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    transition: background 0.3s;
    min-height: 36px;

    &:hover {
      background: #f5f5f5;
    }
  }

  &__text {
    font-size: 16px;
    color: #000;
  }

  &__text--empty {
    text-align: center;
    color: #757575;
    display: block;
    margin-left: auto;
    margin-right: auto;
  }

  &__btn {
    svg {
      width: 15px;
      height: 15px;
      fill: #000;
    }
  }

  &__pin {
    width: 15px;
    height: 15px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  &__favorite {
    width: 15px;
    height: 15px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
}

// // // // // // 

.main-markdown {
  width: 100%;
  display: flex;
  flex-direction: column;

  &__text {
    padding: 10px;
    padding-right: 0;
    display: flex;
    flex-grow: 1;
  }

  &__text-textarea {
    resize: none;
    width: 100%;
    padding-right: 5px;
    font-size: 16px;

    &::-webkit-scrollbar {
      width: 5px;
      background-color: #f5f5f5;
    }

    &::-webkit-scrollbar-thumb {
      border-radius: 10px;
      background-color: #000000;
    }

    &::-webkit-scrollbar-track {
      -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.2);
      border-radius: 10px;
      background-color: #f5f5f5;
    }
  }
}

.main-markdown-top {
  background: #f5f5f5;
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 2px solid #e0e0e0;

  &__wrapper {
    display: flex;
    gap: 10px
  }

  &__row {
    display: flex;

    &-btn:first-child {
      border-top-left-radius: 5px;
      border-bottom-left-radius: 5px;
    }

    &-btn:last-child {
      border-top-right-radius: 5px;
      border-bottom-right-radius: 5px;
    }
  }

  &__row-btn {
    width: 30px;
    height: 28px;
    display: flex;
    justify-content: center;
    align-items: center;
    border: 1px solid #e0e0e0;
    transition: background 0.3s;

    &:hover {
      background: #e0e0e0;
    }
  }
}

// // // // // //

.v-enter-active,
.v-leave-active {
  transition: opacity 0.3s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

// // // // // // 

.deletePopup {
  position: absolute;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 10;
  width: 100%;
  height: 100%;
  background: rgba(#000, .2);
  justify-content: center;
  align-items: center;
  display: flex;
}

.deletePopup-box {
  background: #fff;
  padding: 20px;
  padding-bottom: 10px;
  box-shadow: 0 1px 5px -1px #000;
  border-radius: 5px;

  &__text {
    font-size: 18px;
    margin-bottom: 10px;
  }

  &__subtext {
    font-size: 13px;
    margin-bottom: 15px;
  }

  &__row {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    gap: 10px
  }

  &__row-cancel {
    padding: 5px;
    padding-top: 2px;
    padding-bottom: 2px;
    background-color: #fff;
    border: 1px solid #000;
    border-radius: 5px;
  }

  &__row-sure {
    padding: 5px;
    padding-top: 3px;
    padding-bottom: 3px;
    color: #fff;
    background-color: #e53935;
    border-radius: 5px;
  }
}




@import './assets/_media.scss';
</style>