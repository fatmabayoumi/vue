<template>
  <div class="full-page-container">
    <!-- Header -->
    <header class="app-header">
      <div class="header-left">
        <span class="logo">
          <img src="https://f/b94a41833dbdc8f942daacfef75877d2" alt="Logo" class="logo-img" />
        </span>
        <nav class="main-nav">
          <a href="#">Home</a>
          <a href="#">Expert Board</a>
        </nav>
      </div>
      <div class="header-right">
        <button class="search-btn"><i class="icon-search"></i> Search</button>
        <button class="publish-btn">Publish</button>
      </div>
    </header>

    <!-- Page Content -->
    <div class="page-content-wrapper">
      <div class="content-left-sidebar">
        <div class="context-nav">Discussion / Create Post</div>
      </div>

      <div class="content-main-area">
        <!-- Title -->
        <div class="post-field-group title-field">
          <label class="required-label">Title</label>
          <div class="input-wrapper">
            <input type="text" placeholder="Enter a catchy title" v-model="postTitle" maxlength="200" />
            <span class="char-count">{{ postTitle.length }}/200</span>
          </div>
        </div>

        <!-- Summary -->
        <div class="post-field-group summary-field">
          <label>Summary</label>
          <div class="input-wrapper">
            <textarea placeholder="Enter a brief summary" v-model="postSummary" maxlength="500"></textarea>
            <span class="char-count">{{ postSummary.length }}/500</span>
          </div>
        </div>

        <!-- Rich Text Editor -->
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
                <option value="">Size</option>
                <option value="1">8pt</option>
                <option value="2">10pt</option>
                <option value="3">12pt</option>
                <option value="4">14pt</option>
                <option value="5">18pt</option>
                <option value="6">24pt</option>
                <option value="7">36pt</option>
              </select>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <button
                @click="format('justifyLeft')"
                :class="{ active: isFormatActive('justifyLeft') }"
                title="Align Left"
              >
                ⬅
              </button>
              <button
                @click="format('justifyCenter')"
                :class="{ active: isFormatActive('justifyCenter') }"
                title="Align Center"
              >
                ⬌
              </button>
              <button
                @click="format('justifyRight')"
                :class="{ active: isFormatActive('justifyRight') }"
                title="Align Right"
              >
                ➡
              </button>
              <button @click="format('justifyFull')" :class="{ active: isFormatActive('justifyFull') }" title="Justify">
                ☰
              </button>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <button
                @click="format('insertUnorderedList')"
                :class="{ active: isFormatActive('insertUnorderedList') }"
                title="Bullet List"
              >
                •
              </button>
              <button
                @click="format('insertOrderedList')"
                :class="{ active: isFormatActive('insertOrderedList') }"
                title="Numbered List"
              >
                1.
              </button>
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
              <button @click="insertLink" :class="{ active: isFormatActive('link') }" title="Insert Link">🔗</button>
              <button @click="insertImage" title="Insert Image" :disabled="isUploadingImage">
                {{ isUploadingImage ? '⏳' : '🖼' }}
              </button>
              <button @click="format('formatBlock', '<blockquote>')" :class="{ active: isFormatActive('blockquote') }" title="Quote">"</button>
              <button @click="format('formatBlock', '<pre>')" :class="{ active: isFormatActive('pre') }" title="Code">&lt;/&gt;</button>
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
            <!-- Link Editor Pop-up -->
            <div v-if="showLinkEditor" class="link-editor-popup">
              <input type="text" v-model="currentLinkUrl" placeholder="https://example.com" @keydown.enter.prevent="applyLink" />
              <button @click="applyLink">Apply</button>
              <button @click="removeLink">Remove</button>
              <button @click="showLinkEditor = false">Cancel</button>
            </div>
          </div>
          <div style="position: relative;">
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
              @click="handleEditorClick"
              :data-placeholder="placeholder"
              spellcheck="true"
              aria-label="Rich text editor"
            ></div>
            <!-- Image Resize Handles -->
            <div v-if="selectedImage" class="resize-handles" :style="resizeHandleStyle">
              <div class="resize-handle" @mousedown.prevent="startResize"></div>
            </div>
          </div>
          <div class="status-bar">
            <span>Words: {{ wordCount }}</span>
            <span>Characters: {{ charCount }}</span>
            <span>{{ savedStatus }}</span>
            <span v-if="isUploadingImage" class="upload-status">Uploading image...</span>
          </div>
        </div>

        <!-- Other Form Fields -->
        <div class="form-grid">
          <!-- Attachment -->
          <div class="row">
            <div class="label">附件</div>
            <div class="control">
              <div class="file-btn" @click="triggerFileUpload">
                {{ fileName || '上传附件' }}
              </div>
              <input type="file" ref="fileInput" @change="handleFileChange" style="display: none" />
              <div class="small">信息安全禁止上传个人信息及涉密信息</div>
            </div>
          </div>

          <!-- Domain Selection -->
          <div class="row">
            <div class="label">所属领域 <span class="req">*</span></div>
            <div class="control inline-wrap">
              <label v-for="domain in allDomains" :key="domain" class="checkbox-label">
                <input type="checkbox" :value="domain" v-model="selectedDomains" />
                {{ domain }}
              </label>
            </div>
          </div>

          <!-- Custom Tags -->
          <div class="row">
            <div class="label">自定义标签</div>
            <div class="control">
              <div class="tag-input" contenteditable="true" @keydown.enter.prevent="addTagFromDiv($event)"></div>
              <div class="tags">
                <div class="tag" v-for="(t, i) in customTags" :key="i">
                  {{ t }} <span class="remove" @click="removeTag(i)">×</span>
                </div>
              </div>
              <div class="small">输入自定义标签后按回车完成输入，最多可添加10个用户自定义标签</div>
            </div>
          </div>

          <!-- Category Dropdown -->
          <div class="row">
            <div class="label">Category <span class="req">*</span></div>
            <div class="control">
              <select v-model="selectedCategory" class="select-css">
                <option disabled value="">Please select a category</option>
                <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
              </select>
            </div>
          </div>

          <!-- Expert Name -->
          <div class="row">
            <div class="label">所属专家</div>
            <div class="control">
              <div class="tags">
                  <div class="tag" v-for="(expert, i) in selectedExperts" :key="i">
                  {{ expert }} <span class="remove" @click="removeExpert(i)">×</span>
                </div>
              </div>
              <!-- Available Experts List -->
              <div v-if="availableExperts.length" class="available-experts-list">
                <div
                  v-for="expert in availableExperts"
                  :key="expert"
                  @click="addExpert(expert)"
                  class="available-expert-item"
                >
                  {{ expert }}
                </div>
              </div>
            </div>
          </div>

          <!-- Other Toggles -->
          <div class="row">
            <div class="label">Other</div>
            <div class="control inline">
              <div class="toggle" @click="hasPatent = !hasPatent" :class="{ active: hasPatent }">
                Has Patent {{ hasPatent ? '✅' : '❌' }}
              </div>
              <div class="toggle" @click="hasDemo = !hasDemo" style="margin-left: 10px" :class="{ active: hasDemo }">
                Has Demo {{ hasDemo ? '✅' : '❌' }}
              </div>
            </div>
          </div>

          <!-- Visibility -->
           <div class="row">
            <div class="label">Visibility <span class="req">*</span></div>
            <div class="control inline">
              <div class="toggle-radio" :class="{ active: scope === 'public' }" @click="scope = 'public'">Public</div>
              <div class="toggle-radio" :class="{ active: scope === 'private' }" @click="scope = 'private'">
                Private
              </div>
            </div>
          </div>
        </div>

        <!-- Actions -->
        <div class="actions">
          <div class="btn primary" @click="submitForm">Submit</div>
          <div class="btn" @click="saveDraft">Save Draft</div>
          <div class="btn" @click="cancel">Cancel</div>
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
      postSummary: '',
      fileName: '',
      fileError: '',

      // New dynamic fields data
      categories: ['AI', 'Cloud', 'Software', 'IoT'],
      selectedCategory: '',
      dummyExperts: {
        AI: ['Ali Ahmad / 84029066', 'John Doe / 12345678', 'Jane Smith / 87654321'],
        Cloud: ['Peter Jones / 11223344', 'Susan Miller / 44332211'],
        Software: ['David Chen / 55667788', 'Maria Garcia / 88776655', 'Ali Ahmad / 84029066'],
        IoT: ['Kenji Tanaka / 99887766']
      },
      selectedExperts: [],
      availableExperts: [],

      allDomains: ['信创', '优选', '智改', '节能', '供应', 'EM2.0', '设备安全', '其他'],
      selectedDomains: [],
      customTags: [],
      hasPatent: false,
      hasDemo: false,
      coSubmitters: [],
      scope: 'public',

      // Rich text editor
      content: '',
      isEmpty: true,
      placeholder: 'Start writing your content here...',
      savedStatus: 'Ready',
      autoSaveTimer: null,
      wordCount: 0,
      charCount: 0,
      shortcutHandler: null,
      isUploadingImage: false,
      savedSelection: null,
      activeFormats: {
        bold: false,
        italic: false,
        underline: false,
        strikeThrough: false,
        justifyLeft: false,
        justifyCenter: false,
        justifyRight: false,
        justifyFull: false,
        insertUnorderedList: false,
        insertOrderedList: false,
        blockquote: false,
        pre: false,
        link: false
      },
      showLinkEditor: false,
      currentLinkUrl: '',
      selectedImage: null,

      // API Configuration - Update these with your actual values
      imageUploadApiUrl: 'xxx', // Replace with your actual API endpoint
      imageBaseUrl: 'xx' // Your base URL prefixx
    }
  },
  mounted() {
    this.$nextTick(() => {
      document.execCommand('styleWithCSS', false, true)
      document.execCommand('defaultParagraphSeparator', false, 'p');
      this.loadSavedContent()
      if (this.content) this.$refs.editor.innerHTML = this.content
      this.updateToolbarState()
      this.startAutoSave()
      this.addShortcuts()
      document.addEventListener('click', this.closeDomainDropdown)
    })
  },
  watch: {
    selectedCategory(newCategory) {
      if (newCategory) {
        // When category changes, update the list of available experts and clear selections.
        this.availableExperts = this.dummyExperts[newCategory];
        this.selectedExperts = [];
      } else {
        this.availableExperts = [];
        this.selectedExperts = [];
      }
    }
  },
  computed: {
    resizeHandleStyle() {
      if (!this.selectedImage) return {}
      const editorRect = this.$refs.editor.getBoundingClientRect()
      const imageRect = this.selectedImage.getBoundingClientRect()
      return {
        top: `${imageRect.top - editorRect.top + this.$refs.editor.scrollTop}px`,
        left: `${imageRect.left - editorRect.left}px`,
        width: `${imageRect.width}px`,
        height: `${imageRect.height}px`
      }
    }
  },
  beforeDestroy() {
    if (this.autoSaveTimer) clearInterval(this.autoSaveTimer)
    if (this.shortcutHandler) document.removeEventListener('keydown', this.shortcutHandler)
    document.removeEventListener('click', this.closeDomainDropdown)
    document.removeEventListener('mousemove', this.doResize)
    document.removeEventListener('mouseup', this.stopResize)
  },

  methods: {
    startResize(event) {
        this.initialResize = {
            x: event.clientX,
            y: event.clientY,
            width: this.selectedImage.width,
            height: this.selectedImage.height
        };
        document.addEventListener('mousemove', this.doResize);
        document.addEventListener('mouseup', this.stopResize);
    },
    doResize(event) {
        if (!this.selectedImage || !this.initialResize) return;

        const dx = event.clientX - this.initialResize.x;
        const newWidth = this.initialResize.width + dx;

        if (newWidth > 50) { // Set a minimum width
            this.selectedImage.style.width = `${newWidth}px`;
            this.selectedImage.style.height = 'auto'; // Maintain aspect ratio
        }
    },
    stopResize() {
        document.removeEventListener('mousemove', this.doResize);
        document.removeEventListener('mouseup', this.stopResize);
        this.initialResize = null;
        this.syncFromEditor(); // Save changes
    },
    // --- SELECTION HELPERS ---
    saveSelection() {
      const selection = window.getSelection()
      if (selection.rangeCount > 0) {
        this.savedSelection = selection.getRangeAt(0)
      }
    },
    restoreSelection() {
      if (this.savedSelection) {
        const selection = window.getSelection()
        selection.removeAllRanges()
        selection.addRange(this.savedSelection)
      }
    },
    focusEditor() {
      this.$refs.editor.focus()
      this.restoreSelection()
    },

    // --- UPDATED IMAGE UPLOAD TO API ---
    async uploadImageToApi(file) {
      const formData = new FormData()
      formData.append('image', file)
      formData.append('sourceType', 'fromWord') // Added sourceType as requested

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

        // Prepend the base URL to the returned URL
        return this.imageBaseUrl + data.url
      } catch (error) {
        console.error('Image upload API error:', error)
        throw error
      }
    },

    // --- UPDATED IMAGE INSERTION WITH API UPLOAD ---
    async insertImage() {
      const input = document.createElement('input')
      input.type = 'file'
      input.accept = 'image/*'
      input.onchange = async (e) => {
        const file = e.target.files[0]
        if (!file) return

        // Validate file size (optional - adjust as needed)
        const maxSize = 10 * 1024 * 1024 // 10MB
        if (file.size > maxSize) {
          alert('Image file size must be less than 10MB')
          return
        }

        this.isUploadingImage = true

        try {
          // Upload image to API
          const imageUrl = await this.uploadImageToApi(file)

          // Insert the image with the returned URL
          this.insertHTML(`<img src="${imageUrl}" class="uploaded-image" alt="${file.name}" />`)

          this.savedStatus = 'Image uploaded'
          setTimeout(() => {
            this.savedStatus = 'Ready'
          }, 2000)
        } catch (error) {
          console.error('Image upload failed:', error)
          alert('Could not upload image. Please try again.')
        } finally {
          this.isUploadingImage = false
        }
      }
      input.click()
    },

    clearAllFormatting() {
      if (confirm('Are you sure you want to clear all content and formatting?')) {
        this.$refs.editor.innerHTML = ''
        this.content = ''
        this.syncFromEditor()
        this.$refs.editor.focus()
      }
    },

    handlePaste(event) {
      event.preventDefault()
      this.saveSelection()

      let pasteHtml = ''
      if (event.clipboardData) {
        pasteHtml = event.clipboardData.getData('text/html')
        if (pasteHtml) {
          const sanitizedHtml = this.sanitizeHTML(pasteHtml)
          this.insertHTML(sanitizedHtml)
        } else {
          // Fallback to plain text
          const text = event.clipboardData.getData('text/plain')
          document.execCommand('insertText', false, text)
        }
      }
      this.syncFromEditor()
    },
    sanitizeHTML(html) {
      const doc = new DOMParser().parseFromString(html, 'text/html')
      const body = doc.body

      const allowedTags = [
        'P', 'B', 'I', 'U', 'S', 'STRIKE', 'A', 'UL', 'OL', 'LI', 'BLOCKQUOTE', 'PRE', 'BR', 'DIV', 'SPAN',
        'H1', 'H2', 'H3', 'H4', 'H5', 'H6'
      ]
      const allowedAttrs = ['href', 'target', 'style']

      const walkTheDOM = (node, func) => {
        func(node)
        if (node.childNodes && node.childNodes.length > 0) {
          // Use a static copy of childNodes because the list may be modified during iteration
          Array.from(node.childNodes).forEach(child => {
            walkTheDOM(child, func)
          })
        }
      }

      walkTheDOM(body, (currentNode) => {
        // 1. Remove unwanted nodes (comments, etc.)
        if (currentNode.nodeType !== 1 && currentNode.nodeType !== 3) {
          currentNode.parentNode?.removeChild(currentNode)
          return
        }

        // 2. Process element nodes
        if (currentNode.nodeType === 1) {
            // Remove dangerous tags entirely
            if (['SCRIPT', 'STYLE', 'LINK', 'META', 'IFRAME', 'OBJECT'].includes(currentNode.tagName)) {
                currentNode.parentNode?.removeChild(currentNode)
                return
            }

            // Unwrap disallowed tags, keeping their content
            if (!allowedTags.includes(currentNode.tagName)) {
              currentNode.replaceWith(...currentNode.childNodes)
              return
            }

            // Remove disallowed attributes
            const attrs = Array.from(currentNode.attributes)
            for (const attr of attrs) {
              if (!allowedAttrs.includes(attr.name.toLowerCase())) {
                currentNode.removeAttribute(attr.name)
              }
            }

            // Sanitize style attribute to remove directionality
            if (currentNode.hasAttribute('style')) {
              currentNode.style.direction = ''
              currentNode.style.textAlign = ''
              currentNode.style.unicodeBidi = ''

              // If style attribute is now empty, remove it
              if (!currentNode.getAttribute('style')) {
                currentNode.removeAttribute('style')
              }
            }
            // Remove dir attribute
            if (currentNode.hasAttribute('dir')) {
              currentNode.removeAttribute('dir')
            }
        }
      })

      return body.innerHTML
    },

    // Standard Editor Methods
    format(cmd, value = null) {
      const blockCommands = [
        'justifyLeft', 'justifyCenter', 'justifyRight', 'justifyFull',
        'insertUnorderedList', 'insertOrderedList', 'indent', 'outdent', 'formatBlock'
      ];

      // For block commands, we don't want to restore a granular selection,
      // as it can interfere with how the browser applies the block format.
      // A simple focus is enough.
      if (blockCommands.includes(cmd)) {
        this.$refs.editor.focus();
      } else {
        this.focusEditor(); // For inline commands, restore the exact selection.
      }

      document.execCommand(cmd, false, value);
      this.syncFromEditor();
    },
    isFormatActive(cmd) {
      return this.activeFormats[cmd]
    },
    setFontSize(event) {
      if (event.target.value) {
        this.format('fontSize', event.target.value)
      }
      event.target.value = '' // Reset dropdown
    },
    setTextColor(event) {
      this.format('foreColor', event.target.value)
    },
    setBackgroundColor(event) {
      this.format('hiliteColor', event.target.value)
    },
    insertLink() {
      this.saveSelection() // Save selection before opening the popup
      this.checkSelectionFormats() // Check if we are inside a link to pre-fill
      this.showLinkEditor = true
    },
    applyLink() {
      if (this.currentLinkUrl) {
        // Simple validation for URL
        if (!this.currentLinkUrl.startsWith('http://') && !this.currentLinkUrl.startsWith('https://')) {
          this.currentLinkUrl = 'https://' + this.currentLinkUrl
        }
        this.format('createLink', this.currentLinkUrl)
      }
      this.closeLinkEditor()
    },
    removeLink() {
      this.format('unlink')
      this.closeLinkEditor()
    },
    closeLinkEditor() {
      this.showLinkEditor = false
      this.currentLinkUrl = ''
      this.updateToolbarState()
    },
    handleEditorClick(event) {
      // Ctrl+Click to open links
      const link = event.target.closest('a');
      if (link && (event.ctrlKey || event.metaKey)) {
        event.preventDefault();
        window.open(link.href, '_blank');
        return; // Stop further processing
      }

      // Handle image selection
      if (event.target.tagName === 'IMG') {
        this.selectedImage = event.target
      } else if (this.selectedImage) {
        // Deselect if clicking anywhere else that isn't a resize handle
        if (!event.target.classList.contains('resize-handle')) {
            this.selectedImage = null
        }
      }
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

      this.saveSelection()

      const text = editor.innerText || ''
      this.content = editor.innerHTML
      this.charCount = text.length
      this.wordCount = text.trim() ? text.trim().split(/\s+/).length : 0
      this.isEmpty = !text.trim() && !editor.querySelector('img')
      this.savedStatus = 'Modified'

      this.checkSelectionFormats()
      this.$forceUpdate() // Might still be needed if Vue doesn't pick up deep changes in activeFormats
    },
    checkSelectionFormats() {
      // Reset all formats
      Object.keys(this.activeFormats).forEach(key => {
        this.activeFormats[key] = false
      })

      const selection = window.getSelection()
      if (!selection.rangeCount) return

      let node = selection.anchorNode
      if (!node) return

      // If the node is a text node, start from its parent
      if (node.nodeType === 3) {
        node = node.parentNode
      }

      // Traverse up to the editor root
      while (node && node !== this.$refs.editor) {
        if (node.nodeType === 1) { // Element node
          const el = node
          const style = window.getComputedStyle(el)

          // Inline formats
          if (el.tagName === 'B' || style.fontWeight === 'bold' || parseInt(style.fontWeight, 10) >= 700) this.activeFormats.bold = true
          if (el.tagName === 'I' || style.fontStyle === 'italic') this.activeFormats.italic = true
          if (el.tagName === 'U' || style.textDecorationLine.includes('underline')) this.activeFormats.underline = true
          if (el.tagName === 'S' || el.tagName === 'STRIKE' || style.textDecorationLine.includes('line-through')) this.activeFormats.strikeThrough = true

          // Alignment
          switch (style.textAlign) {
            case 'left': this.activeFormats.justifyLeft = true; break
            case 'center': this.activeFormats.justifyCenter = true; break
            case 'right': this.activeFormats.justifyRight = true; break
            case 'justify': this.activeFormats.justifyFull = true; break
          }

          // Lists
          if (el.tagName === 'UL') this.activeFormats.insertUnorderedList = true
          if (el.tagName === 'OL') this.activeFormats.insertOrderedList = true

          // Block formats
          if (el.tagName === 'BLOCKQUOTE') this.activeFormats.blockquote = true
          if (el.tagName === 'PRE') this.activeFormats.pre = true

          // Link
          if (el.tagName === 'A') {
            this.activeFormats.link = true
            this.currentLinkUrl = el.getAttribute('href')
          }
        }
        node = node.parentNode
      }
    },
    syncFromEditor() {
      this.$nextTick(() => this.updateToolbarState())
    },
    startAutoSave() {
      this.autoSaveTimer = setInterval(() => this.saveContent(), 30000)
    },
    saveContent() {
      localStorage.setItem('richTextContent', this.content)
      this.savedStatus = 'Saved'
      setTimeout(() => {
        if (this.savedStatus === 'Saved') this.savedStatus = 'Ready'
      }, 2000)
    },
    loadSavedContent() {
      const saved = localStorage.getItem('richTextContent')
      if (saved) {
        this.content = saved
        this.syncFromEditor()
      }
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

    // Other Form Methods
    triggerFileUpload() {
      this.$refs.fileInput.click()
    },
    handleFileChange(e) {
      const file = e.target.files[0]
      if (!file) return
      if (file.size > 150 * 1024 * 1024) {
        this.fileError = 'File too large'
        return
      }
      this.fileName = file.name
      this.fileError = ''
    },
    toggleDomain() {
      this.showDomain = !this.showDomain
    },
    selectDomain(d) {
      this.domain = d
      this.showDomain = false
    },
    addTagFromDiv(e) {
      const v = e.target.innerText.trim()
      if (v && !this.customTags.includes(v)) {
        if (this.customTags.length < 10) {
          this.customTags.push(v)
        } else {
          alert('You can add a maximum of 10 custom tags.')
        }
      }
      e.target.innerText = ''
    },
    removeTag(i) {
      this.customTags.splice(i, 1)
    },
    addExpert(expert) {
      if (!this.selectedExperts.includes(expert)) {
        this.selectedExperts.push(expert);
      }
    },
    removeExpert(i) {
        this.selectedExperts.splice(i, 1);
    },
    submitForm() {
      if (!this.selectedCategory) {
        alert('Please select a category')
        return
      }
       if (!this.selectedDomains.length) {
        alert('Please select at least one domain')
        return
      }
      const formData = {
        postTitle: this.postTitle,
        postSummary: this.postSummary,
        content: this.content,
        fileName: this.fileName,
        category: this.selectedCategory,
        domains: this.selectedDomains,
        customTags: this.customTags,
        experts: this.selectedExperts,
        scope: this.scope
      }
      console.log('Form Submitted:', formData)
      alert('Form submitted successfully!')
    },
    saveDraft() {
      this.saveContent()
      alert('Draft saved!')
    },
    cancel() {
      if (confirm('Are you sure? Unsaved changes will be lost.')) {
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
.main-nav a {
  text-decoration: none;
  color: #4a5568;
  font-size: 14px;
  font-weight: 500;
}
.main-nav a:hover {
  color: #2b6cb0;
}
.search-btn,
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
.input-wrapper textarea {
  min-height: 120px;
  resize: vertical;
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
  margin-bottom: 24px;
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
}
.link-editor-popup {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: #fff;
  border: 1px solid #ccc;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  padding: 10px;
  border-radius: 6px;
  display: flex;
  gap: 8px;
  z-index: 100;
}
.link-editor-popup input {
  border: 1px solid #ccc;
  padding: 6px;
  border-radius: 4px;
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
  padding: 0 4px;
  background-color: white;
  cursor: pointer;
}
.color-picker {
  width: 34px;
  padding: 4px;
}
.editor-content {
  min-height: 400px;
  max-height: 600px;
  overflow-y: auto;
  overflow-x: hidden; /* Prevent horizontal scrolling */
  padding: 20px;
  outline: none;
  font-size: 16px;
  line-height: 1.7;
  color: #1a202c;
  position: relative;
  word-wrap: break-word; /* Ensure long words break */
}
.editor-content.empty:before {
  content: attr(data-placeholder);
  color: #a0aec0;
  position: absolute;
  pointer-events: none;
  font-style: italic;
}

/* --- Blockquote & Code Block Styling --- */
.editor-content :deep(blockquote) {
  border-left: 4px solid #e2e8f0;
  margin-left: 0;
  margin-right: 0;
  padding-left: 1.5em;
  color: #718096;
  font-style: italic;
}

.editor-content :deep(pre) {
  background-color: #f7fafc;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  padding: 16px;
  font-family: 'Courier New', Courier, monospace;
  white-space: pre-wrap;
  word-wrap: break-word;
  overflow-x: auto;
}

/* --- FIXED IMAGE STYLING --- */
.editor-content :deep(img),
.editor-content :deep(img.uploaded-image),
.editor-content :deep(img.resizable-image) {
  max-width: 100% !important; /* Prevent images from exceeding container width */
  width: auto !important; /* Allow images to maintain aspect ratio */
  height: auto !important; /* Maintain aspect ratio */
  display: block;
  margin: 16px auto; /* Center the image */
  border-radius: 6px;
  border: 1px solid #e2e8f0;
  box-sizing: border-box; /* Include border in width calculation */
  object-fit: contain; /* Ensure image fits within bounds */
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
}
.row {
  display: grid;
  grid-template-columns: 140px 1fr;
  gap: 16px;
  align-items: start;
}
.label {
  font-weight: 600;
  font-size: 14px;
  color: #2d3748;
  padding-top: 8px;
}
.control.inline-wrap {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  align-items: center;
}
.control.inline {
  display: flex;
  align-items: center;
  gap: 12px;
}
.toggle,
.toggle-radio {
  padding: 10px 12px;
  border: 1px solid #cbd5e0;
  border-radius: 6px;
  cursor: pointer;
  background-color: #fff;
  transition: all 0.2s;
}
.toggle-radio.active,
.toggle.active {
  background-color: #2b6cb0;
  color: #fff;
  border-color: #2b6cb0;
}
.checkbox-label {
  display: flex;
  align-items: center;
  gap: 6px;
  cursor: pointer;
  font-size: 14px;
}
.file-btn {
  padding: 10px 12px;
  border: 1px solid #cbd5e0;
  border-radius: 6px;
  cursor: pointer;
  background-color: #fff;
  transition: all 0.2s;
}
.expert-display {
  padding: 10px 12px;
  background-color: #f7fafc;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  font-size: 14px;
  color: #4a5568;
}
.small {
  font-size: 12px;
  color: #718096;
  margin-top: 4px;
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
.tag-input:focus-within {
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
.select-css {
  display: block;
  width: 100%;
  padding: 10px 12px;
  font-size: 14px;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 6px;
  transition: border-color .15s ease-in-out,box-shadow .15s ease-in-out;
}
.select-css:focus {
  border-color: #80bdff;
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(0,123,255,.25);
}
.available-experts-list {
  margin-top: 10px;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  max-height: 150px;
  overflow-y: auto;
}
.available-expert-item {
  padding: 8px 12px;
  cursor: pointer;
  font-size: 14px;
}
.available-expert-item:hover {
  background-color: #f7fafc;
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
}
.btn.primary:hover {
  background-color: #2c5282;
}
.resize-handles {
  position: absolute;
  border: 2px solid #4299e1;
  pointer-events: none; /* Pass clicks through to the image */
  z-index: 90; /* Below link editor */
}
.resize-handle {
  position: absolute;
  width: 12px;
  height: 12px;
  background-color: #4299e1;
  border: 1px solid #fff;
  border-radius: 2px;
  bottom: -7px;
  right: -7px;
  cursor: nwse-resize;
  pointer-events: all; /* Capture mouse events on the handle */
}
</style>