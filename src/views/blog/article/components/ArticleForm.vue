<template>
  <div class="app-container">
    <el-container>
      <el-header>
        <el-button type="default" size="mini" @click="refreshPage">刷新</el-button>
        <el-button type="primary" size="mini">
          <router-link :to="{path: '/blog/article'}">BackList</router-link>
        </el-button>
      </el-header>
      <el-main>
        <el-form ref="articleFormRef" :model="articleForm" :rules="articleFormRule" size="mini" label-width="100px">
          <el-form-item prop="category_id" label="Category">
            <el-select v-model="articleForm.category_id" clearable filterable required>
              <el-option v-for="item in categoryList" :key="item.id" :label="item.name" :value="item.id" />
            </el-select>
          </el-form-item>
          <el-form-item prop="title" label="Title">
            <el-input v-model="articleForm.title" required />
          </el-form-item>
          <el-form-item prop="slug" label="Slug">
            <el-input v-model="articleForm.slug" />
          </el-form-item>
          <el-form-item label="Tag">
            <el-select v-model="articleForm.tags" clearable multiple filterable>
              <el-option v-for="item in tagList" :key="item.id" :label="item.name" :value="item.id" />
            </el-select>
          </el-form-item>
          <el-form-item prop="source" label="Source">
            <el-input v-model="articleForm.source" required />
          </el-form-item>
          <el-form-item prop="description" label="Description">
            <el-input v-model="articleForm.description" type="textarea" rows="3" />
          </el-form-item>
          <el-form-item prop="is_draft" label="Draft">
            <el-switch v-model="articleForm.is_draft" />
          </el-form-item>
          <el-form-item prop="content" label="Content">
            <tinymce5 ref="contentEditor" v-model="articleForm.content" :height="400" />
          </el-form-item>
        </el-form>
      </el-main>
      <el-footer>
        <el-button v-loading="submitLoading" @click="submitArticleForm">{{ submitButton }}</el-button>
      </el-footer>
    </el-container>
  </div>
</template>

<script>
import Tinymce5 from '@/components/Tinymce5'
import { articleDetail, articleCreate, articleUpdate, tagAll, categoryAll } from '@/api'
const defaultArticleForm = {
  id: undefined,
  category_id: undefined,
  user_id: undefined,
  tags: [],
  is_draft: false,
  view_count: 0,
  title: '',
  slug: '',
  source: '',
  description: '',
  thumbnail: '',
  content: ''
}
export default {
  name: 'ArticleForm',
  components: { Tinymce5 },
  props: {
    isEdit: {
      type: Boolean,
      default: false
    },
    submitButton: {
      type: String,
      default: 'Create'
    }
  },
  data() {
    return {
      submitLoading: false,
      submitLoadingText: '数据提交中...',
      articleForm: {},
      tagList: [],
      categoryList: [],
      articleFormRule: {
        // trigger: 'change'
        title: [{ required: true, trigger: 'blur' }, { max: 225, trigger: 'change' }],
        category_id: [{ required: true, trigger: 'blur' }],
        source: [{ required: true, trigger: 'blur' }]
      }
    }
  },
  created() {
    this.articleForm = Object.assign({}, defaultArticleForm)
    if (this.isEdit === true) {
      const id = this.$route.params && this.$route.params.id
      this.fetchArticleData(id)
    }
    this.getAllTags()
    this.getAllCategories()
  },
  methods: {
    submitArticleForm() {
      this.articleForm.user_id = 1
      this.$refs.articleFormRef.validate(valid => {
        if (valid) {
          this.submitLoading = true
          if (this.isEdit === true) {
            articleUpdate(this.articleForm).then(response => {
              // this.$message.success('successful')
              this.articleForm = Object.assign({}, defaultArticleForm)
            }).catch(error => {
              console.error(error)
            })
          } else {
            articleCreate(this.articleForm).then(response => {
              // this.$message.success('successful')
            })
          }
          this.$message.success(this.submitButton + 'successful')
          this.submitLoading = false
          this.$router.push('/blog/article')
        } else {
          return false
        }
      })
    },
    fetchArticleData(id) {
      articleDetail(id).then(response => {
        this.articleForm = Object.assign(defaultArticleForm, response.data)
      })
    },
    refreshPage() {
      window.location.reload()
    },
    getAllTags() {
      tagAll().then(response => {
        this.tagList = response.data
      })
    },
    getAllCategories() {
      categoryAll().then(response => {
        this.categoryList = response.data
      })
    }
  }
}
</script>