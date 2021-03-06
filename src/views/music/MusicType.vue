<template>
  <v-app>
    <v-container fluid>
      <v-data-iterator
        :items="items"
        :items-per-page.sync="itemsPerPage"
        :page="page"
        :search="search"
        :sort-by="sortBy.toLowerCase()"
        :sort-desc="sortDesc"
        hide-default-footer
      >
        <template v-slot:header>
          <v-toolbar
            dark
            color="blue darken-3"
            class="mb-1"
          >
            <v-text-field
              v-model="search"
              clearable
              flat
              solo-inverted
              hide-details
              label="搜索"
            ></v-text-field>
            <template v-if="$vuetify.breakpoint.mdAndUp">
              <v-spacer></v-spacer>
              <v-select
                v-model="sortBy"
                flat
                solo-inverted
                hide-details
                :items="keys"
                label="按类型排序"
              ></v-select>
              <v-spacer></v-spacer>
              <v-btn-toggle
                v-model="sortDesc"
                mandatory
              >
                <v-btn
                  large
                  depressed
                  color="blue"
                  :value="false"
                >
                  <v-icon>mdi-arrow-up</v-icon>
                </v-btn>
                <v-btn
                  large
                  depressed
                  color="blue"
                  :value="true"
                >
                  <v-icon>mdi-arrow-down</v-icon>
                </v-btn>
              </v-btn-toggle>
            </template>
          </v-toolbar>
        </template>

        <template v-slot:default="props">
          <v-row>
            <v-col
              v-for="item in props.items"
              :key="item.name"
              cols="12"
              sm="6"
              md="4"
              lg="3"
            >
              <v-card>
                <v-card-title class="subheading font-weight-bold">{{ item.name }}</v-card-title>

                <v-divider></v-divider>

                <v-list dense>
                  <v-list-item
                    v-for="(key, index) in filteredKeys"
                    :key="index"
                  >
                    <v-list-item-content :class="{ 'blue--text': sortBy === key }">{{ key }}:</v-list-item-content>
                    <v-list-item-content
                      class="align-end"
                      :class="{ 'blue--text': sortBy === key }"
                    >{{ item[key.toLowerCase()] }}</v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-card>
            </v-col>
          </v-row>
        </template>

        <template v-slot:footer>
          <v-row
            class="mt-2"
            align="center"
            justify="center"
          >
            <span class="grey--text">页类型数</span>
            <v-menu offset-y>
              <template v-slot:activator="{ on }">
                <v-btn
                  dark
                  text
                  color="primary"
                  class="ml-2"
                  v-on="on"
                >
                  {{ itemsPerPage }}
                  <v-icon>mdi-chevron-down</v-icon>
                </v-btn>
              </template>
              <v-list>
                <v-list-item
                  v-for="(number, index) in itemsPerPageArray"
                  :key="index"
                  @click="updateItemsPerPage(number)"
                >
                  <v-list-item-title>{{ number }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>

            <v-spacer></v-spacer>

            <span class="mr-4
            grey--text">
              Page {{ page }} of {{ numberOfPages }}
            </span>
            <v-btn
              fab
              dark
              color="blue darken-3"
              class="mr-1"
              @click="formerPage"
            >
              <v-icon>mdi-chevron-left</v-icon>
            </v-btn>
            <v-btn
              fab
              dark
              color="blue darken-3"
              class="ml-1"
              @click="nextPage"
            >
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
          </v-row>
        </template>
      </v-data-iterator>
    </v-container>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      itemsPerPageArray: [4, 8, 12],
      search: '',
      filter: {},
      sortDesc: false,
      page: 1,
      itemsPerPage: 4,
      sortBy: 'name',
      keys: ['Name', '歌曲数', '歌手数', '粉丝数', '播放量', '热度', '创建时间', '更新时间'],
      items: [
        {
          name: '流行',
          歌曲数: 159,
          歌手数: 6.0,
          粉丝数: 24,
          播放量: 4.0,
          热度: 87,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '华语',
          歌曲数: 237,
          歌手数: 9.0,
          粉丝数: 37,
          播放量: 4.0,
          热度: 129,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '怀旧',
          歌曲数: 262,
          歌手数: 16.0,
          粉丝数: 23,
          播放量: 6.0,
          热度: 337,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '民谣',
          歌曲数: 305,
          歌手数: 37,
          粉丝数: 67,
          播放量: 43,
          热度: 413,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '经典',
          歌曲数: 159,
          歌手数: 6.0,
          粉丝数: 24,
          播放量: 4.0,
          热度: 87,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '夜晚',
          歌曲数: 237,
          歌手数: 9.0,
          粉丝数: 37,
          播放量: 4.0,
          热度: 129,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '摇滚',
          歌曲数: 262,
          歌手数: 16.0,
          粉丝数: 23,
          播放量: 6.0,
          热度: 337,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '伤感',
          歌曲数: 305,
          歌手数: 37,
          粉丝数: 67,
          播放量: 43,
          热度: 413,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '粤语',
          歌曲数: 159,
          歌手数: 6.0,
          粉丝数: 24,
          播放量: 4.0,
          热度: 87,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '治愈',
          歌曲数: 237,
          歌手数: 9.0,
          粉丝数: 37,
          播放量: 4.0,
          热度: 129,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '校园',
          歌曲数: 262,
          歌手数: 16.0,
          粉丝数: 23,
          播放量: 6.0,
          热度: 337,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        },
        {
          name: '日语',
          歌曲数: 303,
          歌手数: 38,
          粉丝数: 67,
          播放量: 43,
          热度: 413,
          创建时间: '1970-01-19 16:53:25',
          更新时间: '1970-01-19 16:53:25'
        }
      ]
    }
  },
  computed: {
    numberOfPages() {
      return Math.ceil(this.items.length / this.itemsPerPage)
    },
    filteredKeys() {
      return this.keys.filter((key) => key !== `Name`)
    }
  },
  methods: {
    nextPage() {
      if (this.page + 1 <= this.numberOfPages) this.page += 1
    },
    formerPage() {
      if (this.page - 1 >= 1) this.page -= 1
    },
    updateItemsPerPage(number) {
      this.itemsPerPage = number
    }
  }
}
</script>

<style scoped lang="scss"></style>
