<template>
  <div
    id="lookUpComponent"
    ref="parentDiv"
    class="custom-lookup-widget"
    :style="lookUpStyles"
    @mouseover="handleMouseOverList"
    @mouseout="handleMouseOutList"
  >
    <select v-model="selectedValue" style="display: none">
      <option v-for="value in selected" :key="value.value" :value="value.value">
        {{ value.text }}
      </option>
    </select>
    <input
      :value="selectedText"
      type="text"
      class="custom-widget-list"
      :style="inputStyles"
      @input="handleInput"
      @keydown="onKeydown"
    />
    <i
      v-if="!disabled"
      class="el-icon-arrow-down icon-arrow"
      @click="toggleList"
    ></i>
    <ul
      :style="listStyles"
      class="search-results"
      @mouseover="handleMouseOverList"
      @mouseout="handleMouseOutList"
    >
      <li
        v-for="option in options"
        ref="selected"
        :key="option.id"
        :title="option.text"
        :full_custom_data="JSON.stringify(option)"
        :value="option.value"
        class="show-li"
        @click="selectOption(option)"
        @mouseenter="handleMouseEnter"
        @mouseleave="handleMouseLeave"
      >
        {{ option.text }}
      </li>
      <li :style="noData"><i class="el-icon-files"></i> 'No data'</li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  props: {
    durl: String,
    dparam: String,
    dwidth: String,
    dlist: String,
  },
  data() {
    return {
      token: null,
      noDataCheck: false,
      disabled: false,
      mouseOver: true,
      searchText: '',
      selectedValue: '',
      selectedText: '',
      old_search_key: '',
      showList: false,
      options: [],
      selected: [],
    }
  },
  computed: {
    // div width
    lookUpStyles() {
      return { width: this.dwidth ? `${parseInt(this.dwidth)}px` : '200px' }
    },

    // input width
    inputStyles() {
      return {
        width: this.dwidth ? `${parseInt(this.dwidth) - 30}px` : '170px',
      }
    },

    // list style
    listStyles() {
      return {
        width: this.dlist ? `${parseInt(this.dlist) + 20}px` : '220px',
        display: this.showList ? 'block' : 'none',
      }
    },
    // no data style
    noData() {
      return {
        display: this.noDataCheck ? 'flex' : 'none',
        height: '100px',
        alignItems: 'center',
        justifyContent: 'center',
        textAlign: 'center',
      }
    },
  },

  methods: {
    // event click on icon
    toggleList() {
      this.mouseOver = true
      document.addEventListener('click', this.handleClick)
      this.searchText = ''
      this.selectedText = ''
      this.getDataList()
    },

    // check mouseover
    handleMouseOverList() {
      this.mouseOver = true
    },

    handleMouseOutList() {
      this.mouseOver = false
    },

    handleClick() {
      if (this.mouseOver === false) {
        this.showList = false
        this.mouseOver = true
      }
    },

    // input event
    handleInput(event) {
      const keyCode = event.which // key-code
      const searchKey = event.target.value // search-key
      this.selectedText = event.target.value
      const thisInput = event.target
      const parentEl = thisInput.parentElement
      const ul = parentEl.querySelector('ul.search-results')
      const showLi = Array.from(ul.querySelectorAll('li.show-li'))

      // if searchKey post to backend
      if (
        searchKey !== '' &&
        (this.old_search_key !== searchKey || keyCode === 8 || keyCode === 46)
      ) {
        this.searchText = event.target.value
        document.addEventListener('click', this.handleClick)
        this.getDataList()
        if (showLi.length > 0) {
          showLi[0].classList.add('selectedLi')
        }
      }
    },

    // =>=>=>=> key down event =>=>=>
    onKeydown(event) {
      const keyCode = event.which // key-code
      const thisInput = event.target
      const parentEl = thisInput.parentElement
      const ul = thisInput.parentElement.querySelector('ul.search-results')
      const showLi = Array.from(ul.querySelectorAll('li.show-li'))
      const selectedLi = ul.querySelector('li.selectedLi')
      let firstLi = ''
      let lastLi = ''
      let thisLi = ''
      if (selectedLi) {
        firstLi = showLi[0].getAttribute('value')
        lastLi = showLi[showLi.length - 1].getAttribute('value')
        thisLi = selectedLi.getAttribute('value')
      }

      if (showLi.length > 0) {
        const ulTopY = ul.getBoundingClientRect().top
        if (keyCode !== 38 && keyCode !== 40) {
          ul.scrollTop = 0
          showLi[0].classList.add('selectedLi')
        }

        if (keyCode === 40 && thisLi !== lastLi) {
          const li = selectedLi.nextElementSibling
          selectedLi.classList.remove('selectedLi')
          li.classList.add('selectedLi')
          li.focus()
          const topLi = li.getBoundingClientRect().top
          const heightLi = li.offsetHeight + 20
          if (topLi - ulTopY > 190) ul.scrollTop += heightLi
        } else if (keyCode === 38 && thisLi !== firstLi) {
          const li = selectedLi.previousElementSibling
          selectedLi.classList.remove('selectedLi')
          li.classList.add('selectedLi')
          const topLi = li.getBoundingClientRect().top
          const heightLi = li.offsetHeight + 20
          if (topLi < ulTopY) ul.scrollTop -= heightLi
          return false
        } else if (thisLi === firstLi) {
          showLi[0].classList.add('selectedLi')
        }
      }

      if (selectedLi) {
        if (selectedLi && keyCode === 13) {
          const fullCustomData = selectedLi.getAttribute('full_custom_data')
          parentEl.setAttribute('full_custom_data', fullCustomData)
          const select = parentEl.querySelector('select')

          if (fullCustomData) {
            const optionValue = JSON.parse(fullCustomData)
            this.selected = []
            this.selected.push(optionValue)
            this.selectedValue = optionValue.value
            this.selectedText = optionValue.text
            this.showList = !this.showList
            document.removeEventListener('click', this.handleClick)
            parentEl.setAttribute('title', optionValue.text)
            thisInput.setAttribute('title', optionValue.text)
            thisInput.value = optionValue.text
            parentEl.dataset.val = optionValue.value
            parentEl.dataset.text_val = optionValue.text
            select.dataset.val = optionValue.value
            select.dataset.text_val = optionValue.text
          }
          ul.scrollTop = 0
          this.showList = false
        }
      }
    },

    // click event on li
    selectOption(option) {
      const el = this.$refs.selected
      const ul = el[0].parentElement
      const parentEl = ul.parentElement
      const fullCustomData = JSON.stringify(option)
      const select = parentEl.querySelector('select')
      const thisInput = parentEl.querySelector('input')

      if (fullCustomData) {
        parentEl.setAttribute('full_custom_data', fullCustomData)
        const optionValue = JSON.parse(fullCustomData)
        this.selected = []
        this.selected.push(optionValue)
        this.selectedValue = optionValue.value
        this.selectedText = optionValue.text
        this.showList = !this.showList
        document.removeEventListener('click', this.handleClick)
        parentEl.setAttribute('title', optionValue.text)
        thisInput.setAttribute('title', optionValue.text)
        thisInput.value = optionValue.text
        parentEl.dataset.val = optionValue.value
        parentEl.dataset.text_val = optionValue.text
        select.dataset.val = optionValue.value
        select.dataset.text_val = optionValue.text
      }
      ul.scrollTop = 0
      this.showList = false
    },

    // add class slectedLi to li
    handleMouseEnter(event) {
      const ul = document.querySelector('ul.search-results')
      const anyLi = ul.querySelectorAll('li')
      anyLi.forEach((li) => li.classList.remove('selectedLi'))
      event.target.classList.add('selectedLi')
    },

    handleMouseLeave(event) {
      event.target.classList.remove('selectedLi')
    },

    // post search key
    getDataList() {
      if (!this.durl) return
      const searchKey = this.searchText.trim()
      const data = {
        search_Key: searchKey,
        typeStr: null,
        clusterSeasonId: null,
      }
      if (this.dparam) {
        Object.assign(data, JSON.parse(this.dparam))
      }

      axios
        .post(`https://192.168.1.55:8443/api/${this.durl}`, data, {
          headers: {
            'Content-type': 'application/json',
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          if (response) {
            this.old_search_key = searchKey
            this.options = response.data.map((obj) => {
              const value = obj.id
              const text = obj.name
              return { value, text }
            })
            if (response.data.length > 0) {
              const parentEl = document.querySelector('#lookUpComponent')
              const ul = parentEl.querySelector('ul.search-results')
              const showLi = Array.from(ul.querySelectorAll('li.show-li'))
              if (showLi.length > 0) {
                showLi[0].classList.add('selectedLi')
              }
              this.noDataCheck = false
            } else {
              this.noDataCheck = true
            }
          }
          this.showList = true
        })
        .catch((error) => {
          this.showList = false
          alert(error)
        })
    },
  },
}
</script>

<style scoped>
.custom-lookup-widget {
  border: 1px solid #000;
  width: 200px;
}
.custom-widget-list {
  width: 170px;
  border: none;
  outline: none;
  height: 25px;
  padding: 2px;
  overflow: hidden;
  text-overflow: ellipsis;
}
.icon-arrow:hover {
  cursor: pointer;
}
.search-results {
  position: absolute;
  z-index: 999;
  background-color: #fff;
  border: 1px solid #ccc;
  max-height: 200px;
  overflow-y: auto;
  list-style: none;
  padding: 0;
  margin: 0;
}
.search-results li {
  font-size: 14px;
  padding: 5px;
  cursor: pointer;
}
.search-results li:hover,
.search-results li.selectedLi {
  background-color: #b7d6f6;
}

.scrollbar-demo-item {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 50px;
  margin: 10px;
  text-align: center;
  border-radius: 4px;
  background: var(--el-color-primary-light-9);
  color: var(--el-color-primary);
}
</style>
