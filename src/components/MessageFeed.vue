<template>
  <div class="message-feed">
    <div
      v-for="message in displayedMessages"
      :key="message.date"
      class="message"
    >
      <a :href="message.authorUrl" target="_blank">{{ message.authorName }}</a>
      - {{ formatDate(message.date) }}
      <p
        :innerHTML="coloredContent(message.content, message.contentPostTones)"
      ></p>
      <hr />
    </div>
    <button @click="loadMore">Загрузить больше</button>
  </div>
</template>

<script>
import feedData from '@/assets/feed.json'

export default {
  data() {
    return {
      messages: [],
      displayedMessages: [],
      perPage: 10,
      currentPage: 1,
    }
  },
  created() {
    // Загружаем данные из импортированного JSON
    this.messages = feedData
    this.displayedMessages = this.messages.slice(0, this.perPage)
  },
  methods: {
    coloredContent(content, tones) {
      let parts = [] // Массив для хранения частей с раскраской
      let lastIndex = 0

      tones.forEach(({ position, tone, length }) => {
        // Добавляем текст до раскрашиваемого участка
        if (position > lastIndex) {
          parts.push(`<span>${content.slice(lastIndex, position)}</span>`)
        }

        // Получаем цвет на основе тональности
        const color = this.getColor(tone)

        // Добавляем раскрашенную часть текста
        parts.push(
          `<span style="background-color:${color}">${content.slice(
            position,
            position + length
          )}</span>`
        )

        // Обновляем индекс последней позиции
        lastIndex = position + length
      })

      // Добавляем оставшуюся часть текста, если она есть
      if (lastIndex < content.length) {
        parts.push(`<span>${content.slice(lastIndex)}</span>`)
      }

      // Возвращаем все части как единый HTML
      return parts.join('')
    },

    getColor(tone) {
      // Возвращаем градиент от красного (-1) к зеленому (1)
      if (tone === 1) return 'green'
      if (tone === 0) return 'yellow'
      if (tone === -1) return 'red'
      const red = Math.round((1 - tone) * 255)
      const green = Math.round((1 + tone) * 255)
      return `rgb(${red}, ${green}, 0)` // Градиент
    },

    applyColor(content, start, length, color) {
      return content.replace(
        new RegExp(`(.{${start}})(.{${length}})`),
        `$1<span style="color:${color}">$2</span>`
      )
    },
    formatDate(dateString) {
      const date = new Date(dateString)
      return date.toLocaleDateString('ru-RU', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
      })
    },

    loadMore() {
      const start = this.currentPage * this.perPage
      const end = start + this.perPage
      this.displayedMessages = this.displayedMessages.concat(
        this.messages.slice(start, end)
      )
      this.currentPage++
    },
  },
}
</script>

<style>
.message-feed {
  max-width: 600px;
  margin: 0 auto;
}
.message {
  margin-bottom: 20px;
}
</style>
