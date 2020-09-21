<template lang="pug">
  div(v-cloak="")
    messages(v-bind:messages="messages")

    div.columns.is-multiline
      div.column.is-12
        a.button.is-primary(v-on:click="createLabel()") 新建标签

      div.column.is-12(v-if="newLabel")
        div.box
          div.columns.is-multiline
            div.column.is-12
              div.tags.has-addons.mb0
                span.tag.is-medium(v-bind:style="{ \
                  color: newLabel.text_color, \
                  backgroundColor: newLabel.background_color \
                }") {{ newLabel.text }}

                span.tag.is-medium
                  kbd {{ shortcutKey(newLabel) | simpleShortcut }}

            div.column
              div.field
                label.label 标签名称
                div.control
                  input.input(
                    type="text"
                    placeholder="输入标签"
                    v-model="newLabel.text"
                  )

            div.column
              div.field
                label.label 快捷键
                div.field.has-addons
                  p.control
                    span.select
                      select(v-model="newLabel.prefix_key")
                        option(value="")
                        option(value="ctrl") Ctrl
                        option(value="shift") Shift
                        option(value="ctrl shift") Ctrl + Shift

                  div.control
                    div.select
                      select(v-model="newLabel.suffix_key")
                        option(disabled="", value="") key
                        option(v-for="ch in shortKeys", v-bind:key="ch") {{ ch }}

            div.column
              div.field
                label.label 选择颜色
                div.field.has-addons
                  div.control
                    div.form__field
                      div.form__input
                        swatches(
                          v-model="newLabel.background_color"
                          colors="basic"
                          show-fallback=true
                          popover-to=""
                          v-bind:trigger-style="{ width: '36px', height: '36px' }"
                        )
                  div.control
                    a.button.random-color-button(
                      v-on:click="setColor(newLabel)"
                    )
                      span.icon.is-small
                        i.fas.fa-sync-alt

            div.column
              div.field
                label.label &nbsp;
                div.field.is-grouped
                  p.control
                    a.button.is-light(v-on:click="cancelCreate()") 取消

                  p.control
                    a.button.is-primary(v-on:click="addLabel()") 创建标签

    div.card
      header.card-header
        p.card-header-title {{ labels.length }} 标签

        a.card-header-icon(href="#", aria-label="more options")
          span.icon
            i.fas.fa-angle-down(aria-hidden="true")

      div.card-content
        div.mb10(v-for="label in labels", v-bind:key="label.id")
          div.level.is-mobile.mb0
            div.level-left
              div.level-item
                p.subtitle.is-5
                  div.tags.has-addons.mb0
                    span.tag.is-medium(v-bind:style="{ \
                      color: label.text_color, \
                      backgroundColor: label.background_color \
                    }") {{ label.text }}

                    span.tag.is-medium
                      kbd {{ shortcutKey(label) | simpleShortcut }}

            div.level-right
              p.level-item
                div.field.is-grouped
                  p.control
                    a.button.is-text(v-on:click="editLabel(label)")
                      span.icon.is-small
                        i.fas.fa-pencil-alt
                      span 编辑

                  p.control
                    a.button.is-text(v-on:click="removeLabel(label)")
                      span.icon.is-small
                        i.fas.fa-trash
                      span 删除

          div.columns(v-show="label === editedLabel")
            div.column
              div.field
                label.label 标签名称
                div.control
                  input.input(
                    type="text"
                    placeholder="输入标签"
                    v-model="label.text"
                  )

            div.column
              div.field
                label.label 快捷键
                div.field.has-addons
                  p.control
                    span.select
                      select(v-model="label.prefix_key")
                        option(value="")
                        option(value="ctrl") Ctrl
                        option(value="shift") Shift
                        option(value="ctrl shift") Ctrl + Shift

                  div.control
                    div.select
                      select(v-model="label.suffix_key")
                        option(disabled="", value="") key
                        option(v-for="ch in shortKeys", v-bind:key="ch") {{ ch }}

            div.column
              div.field
                label.label 选择颜色
                div.field.has-addons
                  div.control
                    div.form__field
                      div.form__input
                        swatches(
                          v-model="label.background_color"
                          colors="basic"
                          show-fallback=true
                          popover-to=""
                          v-bind:trigger-style="{ width: '36px', height: '36px' }"
                        )
                  div.control
                    a.button.random-color-button(
                      v-on:click="setEditColor"
                    )
                      span.icon.is-small
                        i.fas.fa-sync-alt

            div.column
              div.field
                label.label &nbsp;
                div.field.is-grouped
                  p.control
                    a.button.is-light(v-on:click="cancelEdit(label)") 取消

                  p.control
                    a.button.is-primary(v-on:click="doneEdit(label)") 保存更改
</template>

<style scoped>
.random-color-button {
  height: 36px;
  width: 36px;
  background-color: transparent;
  color: #404040;
  border: none;
}
</style>

<script>
import Swatches from 'vue-swatches';
import 'vue-swatches/dist/vue-swatches.min.css';
import HTTP from './http';
import Messages from './messages.vue';
import { simpleShortcut } from './filter';

export default {
  components: { Messages, Swatches },

  filters: { simpleShortcut },

  data: () => ({
    labels: [],
    newLabel: null,
    editedLabel: null,
    messages: [],
    shortKeys: 'abcdefghijklmnopqrstuvwxyz',
  }),

  created() {
    HTTP.get('labels').then((response) => {
      this.labels = response.data;
      this.sortLabels();
    });
  },

  methods: {
    generateColor() {
      const gencolor = Math.floor(Math.random() * 0xFFFFFF).toString(16);
      const randomColor = '#' + ('000000' + gencolor).slice(-6);
      return randomColor;
    },

    blackOrWhite(hexcolor) {
      const r = parseInt(hexcolor.substr(1, 2), 16);
      const g = parseInt(hexcolor.substr(3, 2), 16);
      const b = parseInt(hexcolor.substr(5, 2), 16);
      return ((((r * 299) + (g * 587) + (b * 114)) / 1000) < 128) ? '#ffffff' : '#000000';
    },

    setColor(label) {
      const bgColor = this.generateColor();
      const textColor = this.blackOrWhite(bgColor);
      label.background_color = bgColor;
      label.text_color = textColor;
    },

    setEditColor() {
      this.setColor(this.editedLabel);
    },

    shortcutKey(label) {
      let shortcut = label.suffix_key;
      if (label.prefix_key) {
        shortcut = `${label.prefix_key} ${shortcut}`;
      }
      return shortcut;
    },

    sortLabels() {
      return this.labels.sort((a, b) => ((a.text < b.text) ? -1 : 1));
    },

    addLabel() {
      if (this.newLabel.prefix_key === '') {
        this.newLabel.prefix_key = null;
      }
      HTTP.post('labels', this.newLabel)
        .then((response) => {
          this.cancelCreate();
          this.labels.push(response.data);
          this.sortLabels();
          this.messages = [];
        })
        .catch((error) => {
          console.log(error); // eslint-disable-line no-console
          if (error.response.data.non_field_errors) {
            error.response.data.non_field_errors.forEach((msg) => {
              this.messages.push(msg);
            });
          } else {
            this.messages.push('已存在相同名称的标签或快捷键');
          }
        });
    },

    removeLabel(label) {
      const labelId = label.id;
      HTTP.delete(`labels/${labelId}`).then(() => {
        const index = this.labels.indexOf(label);
        this.labels.splice(index, 1);
      });
    },

    createLabel() {
      this.newLabel = {
        text: '',
        prefix_key: null,
        suffix_key: null,
        background_color: '#209cee',
        text_color: '#ffffff',
      };
      this.setColor(this.newLabel);
    },

    cancelCreate() {
      this.newLabel = null;
    },

    editLabel(label) {
      this.beforeEditCache = Object.assign({}, label);
      this.editedLabel = label;
    },

    doneEdit(label) {
      if (!this.editedLabel) {
        return;
      }
      this.editedLabel = null;
      label.text = label.text.trim();
      if (!label.text) {
        this.removeLabel(label);
      }
      HTTP.patch(`labels/${label.id}`, label)
        .then(() => {
          this.sortLabels();
          this.messages = [];
        })
        .catch((error) => {
          console.log(error); // eslint-disable-line no-console
          if (error.response.data.non_field_errors) {
            error.response.data.non_field_errors.forEach((msg) => {
              this.messages.push(msg);
            });
          } else {
            this.messages.push('已存在相同名称的标签或快捷键');
          }
        });
    },

    cancelEdit(label) {
      this.editedLabel = null;
      Object.assign(label, this.beforeEditCache);
    },
  },
};
</script>
