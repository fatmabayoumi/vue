<template>
  <div class="full-page-container">
    <!-- Header -->
    <header class="app-header">
      <div class="header-left">
        <span class="logo">
          <img src="xxx" alt="Logo" class="logo-img" />
        </span>
        <nav class="main-nav">
          <a href="#">首页</a>
          <a href="#">找专家</a>
        </nav>
      </div>
      <div class="header-right">
        <button class="language-btn">EN / 中</button>
        <button class="publish-btn">发布</button>
      </div>
    </header>

    <!-- Page Content -->
    <div class="page-content-wrapper">
      <div class="content-left-sidebar">
        <div class="context-nav">← 返回项目</div>
      </div>

      <div class="content-main-area">
        <!-- Title -->
        <div class="post-field-group title-field">
          <label class="required-label">* 标题</label>
          <div class="input-wrapper">
            <input
              type="text"
              placeholder="提出一个高品质的讨论话题，最好文字清晰，且从改善需求点，直达技术创新，引入注目..."
              v-model="postTitle"
              maxlength="200"
            />
            <span class="char-count">{{ postTitle.length }}/200</span>
          </div>
        </div>

        <!-- Details / Rich Text Editor -->
        <div class="post-field-group details-field">
          <label class="required-label">* 详情</label>
          <div class="rich-text-editor">
            <div class="toolbar">
              <div class="toolbar-group">
                <button @click="format('bold')" :class="{ active: isFormatActive('bold') }" title="Bold (Ctrl+B)">
                  <strong>B</strong>
                </button>
                <button @click="format('italic')" :class="{ active: isFormatActive('italic') }" title="Italic (Ctrl+I)">
                  <em>I</em>
                </button>
                <button
                  @click="format('underline')"
                  :class="{ active: isFormatActive('underline') }"
                  title="Underline (Ctrl+U)"
                >
                  <u>U</u>
                </button>
                <button
                  @click="format('strikeThrough')"
                  :class="{ active: isFormatActive('strikeThrough') }"
                  title="Strikethrough"
                >
                  <s>S</s>
                </button>
              </div>
              <div class="toolbar-separator"></div>
              <div class="toolbar-group">
                <select @change="setFontSize($event)" class="font-size-select">
                  <option value="">{{ currentFontSize }}</option>
                  <option value="10px">10px</option>
                  <option value="12px">12px</option>
                  <option value="14px">14px</option>
                  <option value="16px">16px</option>
                  <option value="18px">18px</option>
                  <option value="20px">20px</option>
                  <option value="24px">24px</option>
                  <option value="28px">28px</option>
                  <option value="32px">32px</option>
                  <option value="36px">36px</option>
                </select>
              </div>
              <div class="toolbar-separator"></div>
              <div class="toolbar-group">
                <button
                  @click="setAlignment('left')"
                  :class="{ active: currentAlignment === 'left' }"
                  title="Align Left"
                >
                  ⬅
                </button>
                <button
                  @click="setAlignment('center')"
                  :class="{ active: currentAlignment === 'center' }"
                  title="Align Center"
                >
                  ⬌
                </button>
                <button
                  @click="setAlignment('right')"
                  :class="{ active: currentAlignment === 'right' }"
                  title="Align Right"
                >
                  ➡
                </button>
                <button
                  @click="setAlignment('justify')"
                  :class="{ active: currentAlignment === 'justify' }"
                  title="Justify"
                >
                  ☰
                </button>
              </div>
              <div class="toolbar-separator"></div>
              <div class="toolbar-group">
                <button @click="format('outdent')" title="Decrease Indent">⇤</button>
                <button @click="format('indent')" title="Increase Indent">⇥</button>
              </div>
              <div class="toolbar-separator"></div>
              <div class="toolbar-group">
                <input type="color" @change="setTextColor($event)" title="Text Color" class="color-picker" />
                <input
                  type="color"
                  @change="setBackgroundColor($event)"
                  title="Background Color"
                  class="color-picker"
                  value="#ffffff"
                />
              </div>
              <div class="toolbar-separator"></div>
              <div class="toolbar-group">
                <button @click="insertLink" title="Insert Link">🔗</button>
                <button @click="insertImage" title="Insert Image" :disabled="isUploadingImage">
                  {{ isUploadingImage ? '⏳' : '🖼' }}
                </button>
              </div>
              <div class="toolbar-separator"></div>
              <div class="toolbar-group">
                <button @click="format('undo')" title="Undo (Ctrl+Z)">↶</button>
                <button @click="format('redo')" title="Redo (Ctrl+Y)">↷</button>
              </div>
              <div class="toolbar-separator"></div>
              <div class="toolbar-group">
                <button @click="clearAllFormatting" title="Clear Formatting">✖</button>
              </div>
            </div>
            <div
              ref="editor"
              class="editor-content"
              :class="{ empty: isEmpty }"
              contenteditable="true"
              @input="handleInput"
              @keydown="handleKeydown"
              @paste="handlePaste"
              @mouseup="updateToolbarState"
              @keyup="updateToolbarState"
              :data-placeholder="placeholder"
              spellcheck="true"
              aria-label="Rich text editor"
            ></div>
            <div class="status-bar">
              <span>字数: {{ wordCount }}</span>
              <span>字符: {{ charCount }}</span>
              <span>{{ savedStatus }}</span>
              <span v-if="isUploadingImage" class="upload-status">上传中...</span>
            </div>
          </div>
        </div>

        <!-- Other Form Fields -->
        <div class="form-grid">
          <!-- Attachment -->
          <div class="row">
            <div class="label">附件</div>
            <div class="control">
              <div class="file-upload-area">
                <div class="file-btn" @click="triggerFileUpload">
                  {{ fileName || '上传附件' }}
                </div>
                <input type="file" ref="fileInput" @change="handleFileChange" style="display: none" />
                <div class="small">*请上传认证项目的文档，说明文档等。有效期为5分钟，请尽快上传...</div>
                <div class="error" v-if="fileError">{{ fileError }}</div>
              </div>
            </div>
          </div>

          <!-- Project Category -->
          <div class="row">
            <div class="label">所属项目 <span class="req">*</span></div>
            <div class="control">
              <div class="select-div" @click="toggleProject">
                {{ selectedProject || '请选择认证项目' }}
                <span class="dropdown-arrow">▼</span>
                <div class="options" v-show="showProject">
                  <div class="option" v-for="proj in projects" :key="proj.id" @click.stop="selectProject(proj)">
                    {{ proj.name }}
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Custom Tags -->
          <div class="row">
            <div class="label">自定义标签</div>
            <div class="control">
              <div
                class="tag-input"
                contenteditable="true"
                placeholder="输入标签后按回车添加"
                @keydown.enter.prevent="addTagFromDiv($event)"
              ></div>
              <div class="tags">
                <div class="tag" v-for="(t, i) in tags" :key="i">
                  {{ t }} <span class="remove" @click="removeTag(i)">×</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Experts (Improved Chip Design) -->
          <div class="row">
            <div class="label">所属专家</div>
            <div class="control">
              <div class="experts-chip-container">
                <transition-group name="chip-fade" tag="div" class="expert-chips-wrapper">
                  <div class="expert-chip" v-for="(expert, index) in selectedExperts" :key="expert.id">
                    <div class="expert-chip-avatar">
                      {{ getInitials(expert.name) }}
                    </div>
                    <span class="expert-chip-name">{{ expert.name }}</span>
                    <span class="expert-chip-id">{{ expert.id }}</span>
                    <button
                      class="expert-chip-remove"
                      @click="removeExpert(index)"
                      title="移除专家"
                      aria-label="Remove expert"
                    >
                      ×
                    </button>
                  </div>
                </transition-group>
                <div v-if="selectedExperts.length === 0" class="no-experts-message">
                  选择项目后，相关专家将自动显示在这里
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Actions -->
        <div class="actions">
          <div class="btn" @click="cancel">取消</div>
          <div class="btn" @click="saveDraft">保存</div>
          <div class="btn primary" @click="submitForm">提交</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Form data
      postTitle: '',
      fileName: '',
      fileError: '',
      selectedProject: '',
      showProject: false,

      // Projects with associated experts
      projects: [
        {
          id: 1,
          name: '地理信息系统 (GIS)',
          experts: [
            { id: 'EXP-8435695', name: 'Ali Ahmed Abdallsadiq' },
            { id: 'EXP-7234561', name: 'Zhang Wei' },
            { id: 'EXP-9123456', name: 'Maria Garcia' }
          ]
        },
        {
          id: 2,
          name: '人工智能与机器学习',
          experts: [
            { id: 'EXP-5678901', name: 'John Smith' },
            { id: 'EXP-3456789', name: 'Li Ming' },
            { id: 'EXP-8901234', name: 'Sarah Johnson' },
            { id: 'EXP-4567890', name: 'Wang Fang' }
          ]
        },
        {
          id: 3,
          name: '云计算架构',
          experts: [
            { id: 'EXP-2345678', name: 'Chen Jian' },
            { id: 'EXP-6789012', name: 'Emily Brown' }
          ]
        },
        {
          id: 4,
          name: '物联网 (IoT)',
          experts: [
            { id: 'EXP-1234567', name: 'Liu Xiao' },
            { id: 'EXP-7890123', name: 'David Wilson' },
            { id: 'EXP-3456781', name: 'Anna Schmidt' }
          ]
        },
        {
          id: 5,
          name: '网络安全',
          experts: [
            { id: 'EXP-9012345', name: 'Zhang San' },
            { id: 'EXP-4567891', name: 'Michael Davis' }
          ]
        },
        {
          id: 6,
          name: '大数据分析',
          experts: [
            { id: 'EXP-5678902', name: 'Wang Wei' },
            { id: 'EXP-2345679', name: 'Jessica Taylor' },
            { id: 'EXP-8901235', name: 'Carlos Rodriguez' }
          ]
        },
        {
          id: 7,
          name: '区块链技术',
          experts: [
            { id: 'EXP-6789013', name: 'Li Lei' },
            { id: 'EXP-1234568', name: 'Sophie Martin' }
          ]
        },
        {
          id: 8,
          name: '移动应用开发',
          experts: [
            { id: 'EXP-7890124', name: 'Chen Ming' },
            { id: 'EXP-3456782', name: 'Robert Anderson' },
            { id: 'EXP-9012346', name: 'Emma White' }
          ]
        }
      ],

      // Selected experts (will be populated when project is selected)
      selectedExperts: [],

      tags: [],

      // Rich text editor
      content: '',
      isEmpty: true,
      placeholder: '文章形式不拘，但尽量结构清晰的展示研究的分析。观点、想法、并赋予广泛讨论...',
      savedStatus: '准备就绪',
      autoSaveTimer: null,
      wordCount: 0,
      charCount: 0,
      shortcutHandler: null,
      isUploadingImage: false,
      currentAlignment: 'left',
      currentFontSize: '16px',

      // API Configuration
      imageUploadApiUrl: 'xx?',
      imageBaseUrl: 'xx'
    }
  },

  mounted() {
    this.$nextTick(() => {
      this.loadSavedContent()
      if (this.content) this.$refs.editor.innerHTML = this.content
      this.updateToolbarState()
      this.startAutoSave()
      this.addShortcuts()
      document.addEventListener('click', this.closeProjectDropdown)
    })
  },

  beforeDestroy() {
    if (this.autoSaveTimer) clearInterval(this.autoSaveTimer)
    if (this.shortcutHandler) {
      document.removeEventListener('keydown', this.shortcutHandler)
    }
    document.removeEventListener('click', this.closeProjectDropdown)
  },

  methods: {
    // --- EXPERT CHIP METHODS ---
    getInitials(name) {
      if (!name) return '?'
      const parts = name.trim().split(' ')
      if (parts.length >= 2) {
        return (parts[0][0] + parts[1][0]).toUpperCase()
      }
      return name.substring(0, 2).toUpperCase()
    },

    removeExpert(index) {
      this.selectedExperts.splice(index, 1)
    },

    // --- ALIGNMENT FUNCTIONS ---
    setAlignment(align) {
      const editor = this.$refs.editor
      if (!editor) return

      const selection = window.getSelection()
      if (!selection.rangeCount) return

      const range = selection.getRangeAt(0)
      let block = range.commonAncestorContainer

      if (block.nodeType === 3) {
        block = block.parentElement
      }

      while (block && block !== editor && !this.isBlockElement(block)) {
        block = block.parentElement
      }

      if (!block || block === editor) {
        document.execCommand('formatBlock', false, '<div>')
        block = selection.getRangeAt(0).commonAncestorContainer
        if (block.nodeType === 3) {
          block = block.parentElement
        }
      }

      if (block && block !== editor) {
        block.style.textAlign = align
      }

      this.currentAlignment = align
      this.syncFromEditor()
      editor.focus()
    },

    isBlockElement(element) {
      const blockTags = ['DIV', 'P', 'H1', 'H2', 'H3', 'H4', 'H5', 'H6', 'BLOCKQUOTE', 'PRE', 'LI']
      return blockTags.includes(element.tagName)
    },

    // --- IMAGE UPLOAD TO API ---
    async uploadImageToApi(file) {
      const formData = new FormData()
      formData.append('image', file)
      formData.append('sourceType', 'fromWord')

      try {
        const response = await fetch(this.imageUploadApiUrl, {
          method: 'POST',
          credentials: 'include',
          body: formData
        })

        if (!response.ok) {
          const errorBody = await response.text()
          throw new Error(`Upload failed with status: ${response.status}. Response: ${errorBody}`)
        }

        const data = await response.json()
        if (!data.url) {
          throw new Error('API response for image upload is missing the "url" property.')
        }

        return this.imageBaseUrl + data.url
      } catch (error) {
        console.error('Image upload API error:', error)
        throw error
      }
    },

    async insertImage() {
      const input = document.createElement('input')
      input.type = 'file'
      input.accept = 'image/*'
      input.onchange = async (e) => {
        const file = e.target.files[0]
        if (!file) return

        const maxSize = 10 * 1024 * 1024
        if (file.size > maxSize) {
          alert('图片文件大小不能超过10MB')
          return
        }

        this.isUploadingImage = true

        try {
          const imageUrl = await this.uploadImageToApi(file)
          this.insertHTML(`<img src="${imageUrl}" class="uploaded-image" alt="${file.name}" />`)
          this.savedStatus = '图片已上传'
          setTimeout(() => {
            this.savedStatus = '准备就绪'
          }, 2000)
        } catch (error) {
          console.error('Image upload failed:', error)
          alert('无法上传图片，请重试')
        } finally {
          this.isUploadingImage = false
        }
      }
      input.click()
    },

    clearAllFormatting() {
      if (confirm('确定要清除所有内容和格式吗？')) {
        this.$refs.editor.innerHTML = ''
        this.content = ''
        this.syncFromEditor()
        this.$refs.editor.focus()
      }
    },

    handlePaste(event) {
      event.preventDefault()
      const text = (event.clipboardData || window.clipboardData).getData('text/plain')
      document.execCommand('insertText', false, text)
    },

    // Standard Editor Methods
    format(cmd, value = null) {
      document.execCommand(cmd, false, value)
      this.syncFromEditor()
      this.$refs.editor.focus()
    },

    isFormatActive(cmd) {
      if (typeof document.queryCommandState !== 'undefined') {
        try {
          return document.queryCommandState(cmd)
        } catch (e) {
          return false
        }
      }
      return false
    },

    setFontSize(event) {
      if (event.target.value) {
        const editor = this.$refs.editor
        const selection = window.getSelection()
        if (selection.rangeCount > 0) {
          const range = selection.getRangeAt(0)
          const span = document.createElement('span')
          span.style.fontSize = event.target.value
          range.surroundContents(span)
          this.currentFontSize = event.target.value
        }
      }
      event.target.value = ''
      this.syncFromEditor()
    },

    setTextColor(event) {
      this.format('foreColor', event.target.value)
    },

    setBackgroundColor(event) {
      this.format('hiliteColor', event.target.value)
    },

    insertLink() {
      const url = prompt('输入链接地址:')
      if (url) this.format('createLink', url)
    },

    insertHTML(html) {
      document.execCommand('insertHTML', false, html)
      this.syncFromEditor()
    },

    handleInput() {
      this.syncFromEditor()
    },

    handleKeydown(event) {
      if (event.key === 'Tab') {
        event.preventDefault()
        this.format(event.shiftKey ? 'outdent' : 'indent')
      }
    },

    updateToolbarState() {
      const editor = this.$refs.editor
      if (!editor) return

      const text = editor.innerText || ''
      this.content = editor.innerHTML
      this.charCount = text.length
      this.wordCount = text.trim() ? text.trim().split(/\s+/).length : 0
      this.isEmpty = !text.trim() && !editor.querySelector('img')
      this.savedStatus = '已修改'

      const selection = window.getSelection()
      if (selection.rangeCount > 0) {
        let node = selection.anchorNode
        if (node.nodeType === 3) node = node.parentElement
        while (node && node !== editor) {
          const align = window.getComputedStyle(node).textAlign
          if (align && ['left', 'center', 'right', 'justify'].includes(align)) {
            this.currentAlignment = align
          }
          const fontSize = window.getComputedStyle(node).fontSize
          if (fontSize) {
            this.currentFontSize = fontSize
          }
          node = node.parentElement
        }
      }

      this.$forceUpdate()
    },

    syncFromEditor() {
      this.$nextTick(() => this.updateToolbarState())
    },

    startAutoSave() {
      // Auto-save functionality
    },

    saveContent() {
      // Save content functionality
    },

    loadSavedContent() {
      // Load saved content functionality
    },

    addShortcuts() {
      this.shortcutHandler = (e) => {
        if ((e.ctrlKey || e.metaKey) && this.$refs.editor === document.activeElement) {
          switch (e.key.toLowerCase()) {
            case 'b':
              e.preventDefault()
              this.format('bold')
              break
            case 'i':
              e.preventDefault()
              this.format('italic')
              break
            case 'u':
              e.preventDefault()
              this.format('underline')
              break
            case 's':
              e.preventDefault()
              this.saveContent()
              break
          }
        }
      }
      document.addEventListener('keydown', this.shortcutHandler)
    },

    // Form Methods
    triggerFileUpload() {
      this.$refs.fileInput.click()
    },

    handleFileChange(e) {
      const file = e.target.files[0]
      if (!file) return
      if (file.size > 150 * 1024 * 1024) {
        this.fileError = '文件过大'
        return
      }
      this.fileName = file.name
      this.fileError = ''
    },

    toggleProject() {
      this.showProject = !this.showProject
    },

    selectProject(proj) {
      this.selectedProject = proj.name
      this.showProject = false

      // Automatically populate experts when project is selected
      this.selectedExperts = [...proj.experts]
    },

    closeProjectDropdown(e) {
      if (
        this.showProject &&
        this.$el.querySelector('.select-div') &&
        !this.$el.querySelector('.select-div').contains(e.target)
      ) {
        this.showProject = false
      }
    },

    addTagFromDiv(e) {
      const v = e.target.innerText.trim()
      if (v && !this.tags.includes(v)) this.tags.push(v)
      e.target.innerText = ''
    },

    removeTag(i) {
      this.tags.splice(i, 1)
    },

    submitForm() {
      if (!this.selectedProject) {
        alert('请选择所属项目')
        return
      }
      if (!this.postTitle.trim()) {
        alert('请输入标题')
        return
      }

      const formData = {
        postTitle: this.postTitle,
        content: this.content,
        fileName: this.fileName,
        project: this.selectedProject,
        tags: this.tags,
        experts: this.selectedExperts
      }

      console.log('Form Submitted:', formData)
      alert('提交成功!')
    },

    saveDraft() {
      this.saveContent()
      alert('草稿已保存!')
    },

    cancel() {
      if (confirm('确定要取消吗？未保存的更改将丢失。')) {
        localStorage.removeItem('richTextContent')
        location.reload()
      }
    }
  }
}
</script>

<style scoped>
/* --- General & Layout --- */
.full-page-container {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background-color: #f8fafc;
  min-height: 100vh;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 24px;
  height: 64px;
  background-color: #fff;
  border-bottom: 1px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.header-left,
.header-right {
  display: flex;
  align-items: center;
  gap: 20px;
}

.logo-img {
  width: 32px;
  height: 32px;
}

.main-nav {
  display: flex;
  align-items: center;
  gap: 40px;
  margin-left: 60px;
}

.main-nav a {
  text-decoration: none;
  color: #4a5568;
  font-size: 14px;
  font-weight: 500;
}

.main-nav a:hover {
  color: #2b6cb0;
}

.language-btn,
.publish-btn {
  height: 36px;
  padding: 0 16px;
  border-radius: 6px;
  border: 1px solid #cbd5e0;
  background-color: #fff;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
}

.publish-btn {
  background-color: #d0021b;
  color: #fff;
  border-color: #d0021b;
}

.page-content-wrapper {
  display: flex;
  max-width: 1280px;
  margin: 0 auto;
  padding: 24px;
  gap: 24px;
}

.content-left-sidebar {
  width: 200px;
  flex-shrink: 0;
}

.content-main-area {
  width: 100%;
  background-color: #fff;
  padding: 32px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.context-nav {
  font-size: 13px;
  color: #718096;
  margin-bottom: 16px;
  cursor: pointer;
}

.context-nav:hover {
  color: #2b6cb0;
}

/* --- Form Fields --- */
.post-field-group {
  margin-bottom: 24px;
}

.post-field-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  font-size: 14px;
  color: #2d3748;
}

.required-label::before {
  content: '';
}

.input-wrapper {
  position: relative;
}

.input-wrapper input,
.input-wrapper textarea {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #cbd5e0;
  border-radius: 6px;
  font-size: 14px;
  color: #2d3748;
  transition: all 0.2s;
}

.input-wrapper input:focus,
.input-wrapper textarea:focus {
  border-color: #4299e1;
  outline: none;
  box-shadow: 0 0 0 1px #4299e1;
}

.char-count {
  position: absolute;
  right: 10px;
  bottom: 8px;
  font-size: 12px;
  color: #a0aec0;
}

/* --- Rich Text Editor --- */
.rich-text-editor {
  border: 1px solid #cbd5e0;
  border-radius: 8px;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  padding: 12px;
  background-color: #f7fafc;
  border-bottom: 1px solid #e2e8f0;
  gap: 6px;
  border-radius: 8px 8px 0 0;
}

.toolbar-group {
  display: flex;
  gap: 4px;
}

.toolbar-separator {
  width: 1px;
  height: 24px;
  background-color: #e2e8f0;
  margin: 0 8px;
}

.toolbar button {
  width: 34px;
  height: 34px;
  border: 1px solid transparent;
  background: none;
  cursor: pointer;
  border-radius: 6px;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #4a5568;
  transition: all 0.2s;
}

.toolbar button:hover {
  background-color: #e2e8f0;
}

.toolbar button.active {
  background-color: #dbeafe;
  color: #2563eb;
  border-color: #2563eb;
}

.toolbar button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.font-size-select,
.color-picker {
  height: 34px;
  border: 1px solid #cbd5e0;
  border-radius: 6px;
  padding: 0 8px;
  background-color: white;
  cursor: pointer;
  min-width: 80px;
}

.color-picker {
  width: 34px;
  padding: 4px;
  min-width: 34px;
}

.editor-content {
  min-height: 400px;
  max-height: 600px;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 20px;
  outline: none;
  font-size: 16px;
  line-height: 1.7;
  color: #1a202c;
  position: relative;
  word-wrap: break-word;
}

.editor-content.empty:before {
  content: attr(data-placeholder);
  color: #a0aec0;
  position: absolute;
  pointer-events: none;
  font-style: italic;
}

/* --- Image Styling --- */
.editor-content :deep(img),
.editor-content :deep(img.uploaded-image) {
  max-width: 100% !important;
  width: auto !important;
  height: auto !important;
  display: block;
  margin: 16px auto;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
  box-sizing: border-box;
  object-fit: contain;
}

.status-bar {
  display: flex;
  justify-content: space-between;
  padding: 12px 20px;
  background-color: #f7fafc;
  border-top: 1px solid #e2e8f0;
  font-size: 12px;
  color: #718096;
  border-radius: 0 0 8px 8px;
}

.upload-status {
  color: #4299e1;
  font-weight: 600;
}

/* --- Other Form Fields --- */
.form-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
  margin-bottom: 32px;
  margin-top: 32px;
}

.row {
  display: flex;
  align-items: start;
  gap: 16px;
}

.label {
  font-weight: 600;
  font-size: 14px;
  color: #2d3748;
  padding-top: 8px;
  min-width: 100px;
  flex-shrink: 0;
}

.control {
  flex: 1;
}

.file-upload-area {
  width: 100%;
}

.file-btn {
  display: inline-block;
  padding: 10px 16px;
  border: 1px solid #cbd5e0;
  border-radius: 6px;
  cursor: pointer;
  background-color: #fff;
  transition: all 0.2s;
  font-size: 14px;
}

.file-btn:hover {
  background-color: #f7fafc;
  border-color: #4299e1;
}

.select-div {
  position: relative;
  padding: 10px 12px;
  border: 1px solid #cbd5e0;
  border-radius: 6px;
  cursor: pointer;
  background-color: #fff;
  transition: all 0.2s;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.select-div:hover {
  border-color: #4299e1;
}

.dropdown-arrow {
  font-size: 10px;
  color: #a0aec0;
}

.options {
  position: absolute;
  top: 100%;
  left: 0;
  width: 100%;
  margin-top: 4px;
  border: 1px solid #cbd5e0;
  background-color: #fff;
  z-index: 10;
  border-radius: 6px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  max-height: 300px;
  overflow-y: auto;
}

.option {
  padding: 10px 12px;
  cursor: pointer;
  font-size: 14px;
}

.option:hover {
  background-color: #f7fafc;
}

.small {
  font-size: 12px;
  color: #718096;
  margin-top: 6px;
  line-height: 1.5;
}

.error {
  font-size: 12px;
  color: #c53030;
  margin-top: 4px;
}

.tag-input {
  min-height: 40px;
  padding: 8px 12px;
  border: 1px solid #cbd5e0;
  border-radius: 6px;
  background-color: #fff;
  outline: none;
  cursor: text;
}

.tag-input:empty:before {
  content: attr(placeholder);
  color: #a0aec0;
}

.tag-input:focus {
  border-color: #4299e1;
  box-shadow: 0 0 0 1px #4299e1;
}

.tags {
  margin-top: 8px;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tag {
  padding: 6px 12px;
  background-color: #edf2f7;
  border-radius: 16px;
  font-size: 13px;
  color: #4a5568;
  display: flex;
  align-items: center;
}

.tag .remove {
  margin-left: 8px;
  cursor: pointer;
  font-weight: bold;
  color: #a0aec0;
}

.tag .remove:hover {
  color: #c53030;
}

/* --- EXPERT CHIPS (NEW IMPROVED DESIGN) --- */
.experts-chip-container {
  width: 100%;
  min-height: 60px;
  padding: 12px;
  border: 1px solid #cbd5e0;
  border-radius: 8px;
  background-color: #f9fafb;
  transition: all 0.3s ease;
}

.experts-chip-container:hover {
  border-color: #a0aec0;
  background-color: #fff;
}

.expert-chips-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.expert-chip {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px 8px 8px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 24px;
  color: #fff;
  font-size: 13px;
  font-weight: 500;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.25);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  animation: chipSlideIn 0.4s ease-out;
  cursor: default;
  max-width: 100%;
}

@keyframes chipSlideIn {
  from {
    opacity: 0;
    transform: translateY(-10px) scale(0.9);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.expert-chip:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.35);
}

.expert-chip-avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 700;
  color: #fff;
  flex-shrink: 0;
  border: 2px solid rgba(255, 255, 255, 0.5);
  transition: all 0.3s ease;
}

.expert-chip:hover .expert-chip-avatar {
  background-color: rgba(255, 255, 255, 0.4);
  transform: rotate(360deg);
}

.expert-chip-name {
  font-weight: 600;
  letter-spacing: 0.3px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 180px;
}

.expert-chip-id {
  font-size: 11px;
  opacity: 0.85;
  font-weight: 400;
  padding: 2px 6px;
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 10px;
  white-space: nowrap;
}

.expert-chip-remove {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.2);
  border: none;
  color: #fff;
  font-size: 18px;
  font-weight: 700;
  line-height: 1;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 4px;
  flex-shrink: 0;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  padding: 0;
}

.expert-chip-remove:hover {
  background-color: rgba(239, 68, 68, 0.9);
  transform: rotate(90deg) scale(1.15);
}

.expert-chip-remove:active {
  transform: rotate(90deg) scale(0.95);
}

/* Chip fade transition */
.chip-fade-enter-active {
  animation: chipSlideIn 0.4s ease-out;
}

.chip-fade-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 1, 1);
}

.chip-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px) scale(0.8);
}

.no-experts-message {
  color: #a0aec0;
  font-size: 13px;
  font-style: italic;
  text-align: center;
  padding: 16px;
}

.req {
  color: #c53030;
}

/* --- Actions --- */
.actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid #e2e8f0;
}

.btn {
  display: inline-block;
  padding: 12px 24px;
  border-radius: 6px;
  border: 1px solid #cbd5e0;
  cursor: pointer;
  background-color: #fff;
  font-weight: 600;
  font-size: 14px;
  transition: all 0.2s;
}

.btn.primary {
  background-color: #2b6cb0;
  color: #fff;
  border-color: #2b6cb0;
}

.btn:hover {
  border-color: #a0aec0;
  background-color: #f7fafc;
}

.btn.primary:hover {
  background-color: #2c5282;
}
</style>
