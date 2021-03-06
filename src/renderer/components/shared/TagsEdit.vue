<template>
  <section>
    <slot />
    <b-sidebar
      v-model="showModal"
      type="is-white"
      :fullheight="true"
      :overlay="true"
      :right="true"
      :can-cancel="['escape']"
      class="is-medium"
    >
      <div class="section">
        <b-field label="Tags">
          <b-autocomplete
            v-model="keyword"
            placeholder="e.g. Productivity"
            :open-on-focus="true"
            :data="data"
            field="name"
            :clearable="true"
            @select="(option) => (selected = option)"
          >
            <template slot-scope="props">
              <div class="media">
                <div class="media-left">
                  <b-icon
                    icon="circle"
                    :type="props.option.type"
                    size="is-small"
                  />
                </div>
                <div class="media-content">
                  {{ props.option.name }}
                </div>
              </div>
            </template>
          </b-autocomplete>
        </b-field>
        <div v-if="isSelected">
          <b-field label="Color">
            <b-dropdown v-model="selected.type" aria-role="list">
              <button slot="trigger" class="button" type="button">
                <template>
                  <b-icon icon="circle" :type="selected.type" />
                  <span class="pl-1">{{ getNameByType(selected.type) }}</span>
                </template>
                <b-icon icon="menu-down" />
              </button>

              <b-dropdown-item
                v-for="(type, index) in types"
                :key="index"
                :value="type"
                aria-role="listitem"
              >
                <div class="media">
                  <b-icon class="media-left" icon="circle" :type="type" />
                  <div class="media-content">
                    <span class="is-size-6">{{ getNameByType(type) }}</span>
                  </div>
                </div>
              </b-dropdown-item>
            </b-dropdown>
          </b-field>
          <b-field label="Description">
            <b-input
              v-model="selected.description"
              placeholder="e.g. great label"
            />
          </b-field>
        </div>
        <div class="footer">
          <div class="buttons">
            <b-button
              v-if="isSelected"
              type="is-primary"
              :loading="saveButtonLoading"
              @click="updateTag"
            >
              Save
            </b-button>
            <b-button @click="toggle">Cancel</b-button>
          </div>
        </div>
      </div>
    </b-sidebar>
  </section>
</template>

<script lang="ts">
import { computed, defineComponent, ref } from '@nuxtjs/composition-api'
import Tag from '~/models/Tag'
import Helpers from '~/plugins/helpers'

export default defineComponent({
  name: 'TagsEdit',
  setup (props, { emit }) {
    const showModal = ref(false)
    const keyword = ref(null)
    const selected = ref<Tag>(null)
    const types = ref(['is-primary', 'is-danger', 'is-success', 'is-light'])
    const saveButtonLoading = ref(false)

    const toggle = () => {
      showModal.value = !showModal.value
    }

    const data = computed(() => {
      let q = Tag.query()

      if (keyword.value !== null && keyword.value !== '') {
        q = q.search(keyword.value)
      }

      return q.get()
    })

    const isSelected = computed(() => selected.value)

    const getNameByType = type => {
      const data = {
        'is-primary': 'Blue',
        'is-danger': 'Red',
        'is-success': 'Green',
        'is-light': 'Grey'
      }
      return data[type]
    }

    const updateTag = async () => {
      saveButtonLoading.value = true
      await Tag.update({
        where: selected.value.id,
        data: {
          type: selected.value.type,
          description: selected.value.description
        }
      }).finally(async () => {
        await Helpers.delay(200)
        saveButtonLoading.value = false
        toggle()
      })
    }

    return {
      showModal,
      data,
      keyword,
      selected,
      types,
      isSelected,
      saveButtonLoading,
      toggle,
      getNameByType,
      updateTag
    }
  }
})
</script>
