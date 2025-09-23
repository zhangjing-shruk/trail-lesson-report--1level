<script setup>
import { ref } from 'vue'
import html2canvas from 'html2canvas'

// 响应式数据
const currentPage = ref('student') // 直接显示学生报告页面
const studentName = ref('')
const currentDate = ref('')
const classContent = ref('')
const teacherEvaluation = ref('')
const isEditingName = ref(false)
const tempStudentName = ref('')
const isExporting = ref(false)
const isRefreshing = ref(false)

// 可编辑的评分状态
const editableRatings = ref({
  vocabulary: 0,
  grammar: 0,
  reading: 0,
  writing: 0
})

// 可编辑的建议文案（仅教材4）
const editableSuggestions = ref('المفردات: الاستمرار في الاستبدال المرادف وتراكم العبارات المتقدمة، مع التركيز على التعبير المكتوب والمجرد.\n\nالقواعد: تنظيم منهجي للجمل الطويلة الصعبة، وصقل التعبير المعقد.\n\nالقراءة: تحليل عميق للهيكل والعلاقات المنطقية، وتحسين قدرة التلخيص والاستنتاج.\n\nالكتابة: تنظيم هيكل المقالات الجدلية.')

// 建议文案编辑状态
const isEditingSuggestions = ref(false)
const tempSuggestions = ref('')

// 维度星级文案数据（仅教材4）
const dimensionTexts = ref({
  vocabulary: [
    'يحتاج إلى الاستمرار في تراكم العبارات المتقدمة والتعبير المجرد.',
    'يعرف المفردات المتقدمة ويبدأ في استخدام التعبيرات المجردة.',
    'لديه مخزون ممتاز من المفردات ويستخدم التعبيرات المتقدمة بثقة.',
    'يتقن المفردات المعقدة ويستخدم الاستبدال المرادف بمهارة.',
    'يستخدم مفردات متطورة جداً بدقة تامة في جميع السياقات.'
  ],
  grammar: [
    'يحتاج إلى تنظيم منهجي للجمل الطويلة الصعبة وصقل التعبير المعقد.',
    'يفهم القواعد المعقدة ويبدأ في تطبيقها في جمل طويلة.',
    'يطبق القواعد المتقدمة في تراكيب معقدة ومتنوعة.',
    'يتقن التراكيب المعقدة ويستخدمها بطلاقة في التعبير.',
    'لديه إتقان كامل للقواعد المعقدة ويطبقها بشكل إبداعي.'
  ],
  reading: [
    'يحتاج إلى تحليل عميق للهيكل والعلاقات المنطقية وتحسين قدرة التلخيص.',
    'يفهم النصوص المعقدة ويبدأ في التحليل العميق للهيكل.',
    'يحلل النصوص بعمق ويفهم العلاقات المنطقية المعقدة.',
    'يقرأ بفهم متقدم ويلخص الأفكار الرئيسية بدقة.',
    'يحلل النصوص تحليلاً نقدياً شاملاً ويستنتج المعاني العميقة.'
  ],
  writing: [
    'يحتاج إلى تنظيم هيكل المقالات الجدلية وتطوير الحجج.',
    'يكتب مقالات منظمة ولكن يحتاج إلى تحسين البنية الجدلية.',
    'يكتب مقالات جدلية مقنعة مع حجج قوية ومنطقية.',
    'يكتب بأسلوب متقدم مع تنظيم ممتاز للأفكار والحجج.',
    'يكتب مقالات جدلية متطورة بأسلوب إبداعي ومقنع.'
  ]
})

// 维度文案编辑状态
const editingDimensionText = ref({
  dimension: null,
  rating: null
})

// 教材4的固定信息
const textbookInfo = {
  name: 'Your passport to STEP success I',
  color: '#9C27B0'
}

// 预设内容数据（仅保留教材4的单词和阅读技巧）
const presetContent = {
  words: [
    { id: 1, word: 'cruel', detail: '(كلمات من الامتحانات الحقيقية)' },
    { id: 2, word: 'brutality', detail: '(كلمات من الامتحانات الحقيقية)' },
    { id: 3, word: 'presence', detail: '(كلمات من الامتحانات الحقيقية)' },
    { id: 4, word: 'parade', detail: '(كلمات من الامتحانات الحقيقية)' },
    { id: 5, word: 'forbid', detail: '(كلمات من الامتحانات الحقيقية)' },
    { id: 6, word: 'prohibit', detail: '(كلمات من الامتحانات الحقيقية)' },
    { id: 7, word: 'parliament', detail: '(كلمات من الامتحانات الحقيقية)' },
    { id: 8, word: 'legislative', detail: '(كلمات من الامتحانات الحقيقية)' }
  ],
  readingSkills: [
    { id: 1, skill: 'طريقة تجوز الفواصل', description: '① هذه المهارة في القراءة تشبه القوافل في الصحراء: عليك تمييز الطريق الرئيسي عن "المسارات الجانبية". بعد الفواصل غالبًا تأتي تفاصيل إضافية، لذا تخطيها يساعدك على التركيز على الفكرة الرئيسية وعدم التشتت بالتفاصيل. هذه الطريقة مفيدة جدًا مع الجمل الطويلة والمعقدة.\n② تساعد الطلاب على فهم الجمل الطويلة والصعبة، وعدم الانشغال بالتفاصيل الصغيرة، وتقلل القلق أثناء القراءة، وتطور مهارة المسح السريع للنصوص.' }
  ]
}



// 生成星级显示
const generateStars = (rating) => {
  const stars = []
  for (let i = 1; i <= 5; i++) {
    stars.push(`<span class="star ${i <= rating ? 'active' : 'inactive'}">★</span>`)
  }
  return stars.join('')
}

// 点击星星更新评分
const updateRating = (dimension, rating) => {
  editableRatings.value[dimension] = rating
}

// 格式化建议文案用于显示
const formatSuggestionsForDisplay = (text) => {
  if (!text) return '<p class="placeholder-text">انقر هنا لتحرير توصيات...</p>'
  return text.split('\n\n').filter(line => line.trim()).map(line => `<p>${line}</p>`).join('')
}

// 开始编辑建议文案
const startEditingSuggestions = (event) => {
  isEditingSuggestions.value = true
  const element = event.target
  element.classList.add('editing')
  // 如果是占位符文本，清空内容
  if (element.innerHTML.includes('placeholder-text')) {
    element.innerHTML = ''
  }
}

// 更新临时建议文案
const updateTempSuggestions = (event) => {
  tempSuggestions.value = event.target.innerText
}

// 保存建议文案
const saveSuggestions = (event) => {
  const content = event.target.innerText.trim()
  editableSuggestions.value = content
  isEditingSuggestions.value = false
  event.target.classList.remove('editing')
  
  // 重新格式化显示内容
  event.target.innerHTML = formatSuggestionsForDisplay(content)
  
  // 建议文案保存，不显示提示
}

// 格式化维度文案用于显示
const formatDimensionTextForDisplay = (text) => {
  if (!text) return '<p class="placeholder-text">انقر هنا لتحرير التقييم التفصيلي لكل جانب...</p>'
  return `<p>${text}</p>`
}

// 开始编辑维度文案
const startEditingDimensionText = (event) => {
  editingDimensionText.value = { editing: true }
  const element = event.target
  element.classList.add('editing')
  
  // 直接进入编辑模式，不修改现有的HTML结构
  // 因为模板已经正确生成了段落结构
}

// 更新临时维度文案
const updateTempDimensionText = (event) => {
  // 实时更新，无需特殊处理
}

// 保存维度文案
const saveDimensionText = (event) => {
  const content = event.target.innerText.trim()
  if (content) {
    // 将内容按行分割并更新到对应维度
    const lines = content.split('\n\n').filter(line => line.trim())
    let index = 0
    Object.keys(editableRatings.value).forEach(key => {
      if (editableRatings.value[key] > 0 && lines[index]) {
        dimensionTexts.value[key][editableRatings.value[key] - 1] = lines[index].trim()
        index++
      }
    })
  }
  editingDimensionText.value = { editing: false }
  event.target.classList.remove('editing')
  
  // 维度评价保存，不显示提示
}

// 获取维度名称
const getDimensionName = (key) => {
  const dimensionNames = {
    vocabulary: 'المفردات',
    grammar: 'القواعد', 
    reading: 'القراءة',
    writing: 'الكتابة'
  }
  return dimensionNames[key] || key
}

// 等待字体加载完成
const waitForFonts = async () => {
  if ('fonts' in document) {
    try {
      await document.fonts.ready
      // 额外等待一点时间确保字体完全渲染
      await new Promise(resolve => setTimeout(resolve, 500))
    } catch (error) {
      console.warn('字体加载检测失败:', error)
    }
  } else {
    // 如果不支持字体API，等待固定时间
    await new Promise(resolve => setTimeout(resolve, 1000))
  }
}


// 导出为图片
const exportAsImage = async () => {
  if (isExporting.value) return
  
  try {
    isExporting.value = true
    
    console.log('=== 开始导出图片 ===')
    
    // 获取要导出的元素
    const element = document.getElementById('report-content')
    if (!element) {
      console.error('未找到报告内容元素')
      alert('لا يمكن العثور على محتوى التقرير')
      return
    }
    
    console.log('找到导出元素:', element)
    console.log('元素尺寸:', {
      offsetWidth: element.offsetWidth,
      offsetHeight: element.offsetHeight,
      scrollWidth: element.scrollWidth,
      scrollHeight: element.scrollHeight
    })
    
    // 滚动到顶部
    window.scrollTo(0, 0)
    
    // 等待字体加载
    console.log('等待字体加载...')
    await new Promise(resolve => setTimeout(resolve, 500))
    
    // 设置最初的简单字体配置
    const allElements = element.querySelectorAll('*')
    const fontFamily = 'IBM Plex Sans Arabic, Noto Sans Arabic, Arial, sans-serif'
    allElements.forEach(el => {
      el.style.fontFamily = fontFamily
    })
    element.style.fontFamily = fontFamily
    
    // 等待更长时间确保字体和渲染完成
    console.log('等待字体渲染完成...')
    await new Promise(resolve => setTimeout(resolve, 1000))
    
    console.log('开始html2canvas截图...')
    
    // 使用html2canvas，保持原有UI设计
    const canvas = await html2canvas(element, {
      backgroundColor: '#ffffff',
      logging: true,
      useCORS: true,
      allowTaint: true,
      scale: 2,
      width: element.scrollWidth,
      height: element.scrollHeight,
      scrollX: 0,
      scrollY: 0
    })
    
    console.log('html2canvas完成')
    console.log('Canvas尺寸:', canvas.width, 'x', canvas.height)
    
    // 创建下载链接
    const link = document.createElement('a')
    link.download = `تقرير_التقييم_${studentName.value}_${currentDate.value.replace(/\//g, '-')}.png`
    link.href = canvas.toDataURL('image/png')
    
    // 触发下载
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    
    // 显示成功提示
    showSuccessMessage('تم تصدير التقرير كصورة بنجاح!')
  } catch (error) {
    console.error('فشل في تصدير الصورة:', error)
    alert('فشل في تصدير الصورة، يرجى المحاولة مرة أخرى')
  } finally {
    isExporting.value = false
  }
}

// 模拟CRM系统数据获取
const fetchCRMData = () => {
  // 模拟从CRM系统获取学生信息
  const mockStudents = ['张小明', '李小红', '王小华', '刘小强', '陈小美']
  const randomStudent = mockStudents[Math.floor(Math.random() * mockStudents.length)]
  
  studentName.value = randomStudent
  currentDate.value = new Date().toLocaleDateString('zh-CN')
}

// 编辑学生姓名
const startEditName = () => {
  isEditingName.value = true
  tempStudentName.value = studentName.value
}

const saveStudentName = () => {
  if (tempStudentName.value.trim()) {
    studentName.value = tempStudentName.value.trim()
  }
  isEditingName.value = false
}

const cancelEditName = () => {
  isEditingName.value = false
  tempStudentName.value = studentName.value
}

// 刷新CRM数据
const refreshCRMData = async () => {
  if (isRefreshing.value) return
  
  try {
    isRefreshing.value = true
    // 模拟网络延迟
    await new Promise(resolve => setTimeout(resolve, 800))
    fetchCRMData()
    showSuccessMessage('تم تحديث معلومات الطالب من نظام CRM')
  } finally {
    isRefreshing.value = false
  }
}

// 显示成功消息
const showSuccessMessage = (message) => {
  // 创建临时提示元素
  const toast = document.createElement('div')
  toast.textContent = message
  toast.style.cssText = `
    position: fixed;
    top: 20px;
    right: 20px;
    background: #4CAF50;
    color: white;
    padding: 12px 24px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    z-index: 1000;
    font-weight: bold;
    animation: slideIn 0.3s ease;
  `
  
  // 添加动画样式
  const style = document.createElement('style')
  style.textContent = `
    @keyframes slideIn {
      from { transform: translateX(100%); opacity: 0; }
      to { transform: translateX(0); opacity: 1; }
    }
  `
  document.head.appendChild(style)
  
  document.body.appendChild(toast)
  
  // 3秒后移除
  setTimeout(() => {
    toast.remove()
    style.remove()
  }, 3000)
}



// 初始化时加载默认内容
fetchCRMData()
</script>

<template>
  <div class="app">
    <!-- 报告页面 -->
    <div class="container">
      <!-- 头部标题 -->
      <header class="header">
        <h1>تقرير تقييم مستوى اللغة الإنجليزية – أكاديمية 51Talk</h1>
      </header>

      <!-- 报告预览区域 -->
      <div class="report-preview" id="report-content">
        <div class="report-header">
          <h2>تقرير تقييم مستوى اللغة الإنجليزية – أكاديمية 51Talk</h2>
          <div class="report-info">
            <div class="info-item">
               <span class="label">اسم الطالب:</span>
               <span v-if="!isEditingName" class="value editable" @click="startEditName">
                 {{ studentName }} ✏️
               </span>
               <div v-else class="edit-name">
                 <input 
                   v-model="tempStudentName" 
                   @keyup.enter="saveStudentName"
                   @keyup.escape="cancelEditName"
                   class="name-input"
                   autofocus
                 />
                 <button @click="saveStudentName" class="save-btn">✓</button>
                 <button @click="cancelEditName" class="cancel-btn">✗</button>
               </div>
             </div>
            <div class="info-item">
              <span class="label">تاريخ الدرس:</span>
              <span class="value">{{ currentDate }}</span>
            </div>
            <div class="info-item">
              <span class="label">مادة الدرس:</span>
              <span class="value">{{ textbookInfo.name }}</span>
            </div>

          </div>
        </div>

        <div class="report-content">
          <!-- 课堂内容汇总模块 -->
          <div class="content-section">
            <h3>📚 من خلال درس التجربة هذا، لقد تعلمت المحتوى التالي، رائع جدًا!</h3>
            <div class="content-box">
              <!-- 掌握单词 -->
              <div class="learning-item">
                <h4>عدد الكلمات التي تعلمتها في هذا الدرس {{ presetContent.words?.length || 0 }} كلمات</h4>
                <div class="word-list">
                  <div v-for="(word, index) in presetContent.words || []" :key="word.id" class="word-item">
                    <div class="content-left">
                      <span class="word-number">{{ index + 1 }}️⃣</span>
                      <span class="word-text">{{ word.word }} <span class="word-detail">{{ word.detail }}</span></span>
                    </div>
                  </div>
                </div>
              </div>

              <!-- 掌握阅读技巧 -->
              <div class="learning-item">
                <h4>مهارة القراءة التي تعلمتها في هذا الدرس</h4>
                <div class="skill-list">
                  <div v-for="(skill, index) in presetContent.readingSkills || []" :key="skill.id" class="skill-item">
                    <div class="skill-content">
                      <div class="content-left">
                        <strong>{{ skill.skill }}</strong>
                      </div>
                      <div v-if="skill.description" class="skill-description" style="white-space: pre-line;">{{ skill.description }}</div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- 教师评价模块 -->
          <div class="content-section">
            <h3>👨‍🏫 تقييم المعلم</h3>
            <div class="content-box">
              <!-- 维度评级 -->
              <div class="evaluation-ratings">
                <h4>تقييم كل جانب</h4>
                <div class="rating-grid">
                  <div class="rating-item">
                    <span class="rating-label">المفردات:</span>
                    <div class="rating-stars">
                      <span 
                        v-for="star in 5" 
                        :key="star" 
                        class="star clickable-star" 
                        :class="{ 'active': star <= editableRatings.vocabulary, 'inactive': star > editableRatings.vocabulary }"
                        @click="updateRating('vocabulary', star)"
                      >★</span>
                    </div>
                  </div>
                  <div class="rating-item">
                    <span class="rating-label">القواعد:</span>
                    <div class="rating-stars">
                      <span 
                        v-for="star in 5" 
                        :key="star" 
                        class="star clickable-star" 
                        :class="{ 'active': star <= editableRatings.grammar, 'inactive': star > editableRatings.grammar }"
                        @click="updateRating('grammar', star)"
                      >★</span>
                    </div>
                  </div>
                  <div class="rating-item">
                    <span class="rating-label">القراءة:</span>
                    <div class="rating-stars">
                      <span 
                        v-for="star in 5" 
                        :key="star" 
                        class="star clickable-star" 
                        :class="{ 'active': star <= editableRatings.reading, 'inactive': star > editableRatings.reading }"
                        @click="updateRating('reading', star)"
                      >★</span>
                    </div>
                  </div>
                  <div class="rating-item">
                    <span class="rating-label">الكتابة:</span>
                    <div class="rating-stars">
                      <span 
                        v-for="star in 5" 
                        :key="star" 
                        class="star clickable-star" 
                        :class="{ 'active': star <= editableRatings.writing, 'inactive': star > editableRatings.writing }"
                        @click="updateRating('writing', star)"
                      >★</span>
                    </div>
                  </div>
                </div>
              </div>

              <!-- 维度文案显示 -->
              <div class="evaluation-section">
                <h4>التقييم التفصيلي لكل جانب</h4>
                <div 
                  v-if="Object.values(editableRatings).some(rating => rating > 0)"
                  class="evaluation-text editable-content"
                  contenteditable="true"
                  @blur="saveDimensionText"
                  @focus="startEditingDimensionText"
                  @input="updateTempDimensionText"
                >
                  <p v-for="(dimension, key) in editableRatings" :key="key">
                    <span v-if="dimension > 0">
                      <strong>{{ getDimensionName(key) }}:</strong> 
                      <span v-html="formatDimensionTextForDisplay(dimensionTexts[key][dimension - 1])"></span>
                    </span>
                  </p>
                </div>
              </div>

              <!-- 后续建议 -->
              <div class="evaluation-section">
                <h4>توصيات</h4>
                <div 
                  class="evaluation-text editable-content"
                  contenteditable="true"
                  @blur="saveSuggestions"
                  @focus="startEditingSuggestions"
                  @input="updateTempSuggestions"
                  v-html="formatSuggestionsForDisplay(editableSuggestions || '')"
                ></div>
              </div>
            </div>
          </div>

          <!-- 等级展示模块 -->
          <div class="content-section">
            <h3>🎯 خطة الامتحان المستهدف</h3>

            <div class="content-box level-display">
              <div class="current-level">
                <div class="level-chart">
                  <table class="chart-table">
                    <thead>
                      <tr class="chart-row chart-header">
                        <th class="chart-cell row-label-header">الامتحان المستهدف</th>
                        <th class="chart-cell level1-header">المرحلة الإعدادية</th>
                        <th class="chart-cell level2-header">الامتحان النهائي للفصل</th>
                        <th class="chart-cell level3-header">امتحان التخرج</th>
                        <th class="chart-cell level4-header">STEP</th>
                        <th class="chart-cell level5-header">IELTS</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr class="chart-row">
                        <td class="chart-cell row-label">درجة IELTS المعادلة</td>
                        <td class="chart-cell">≤4.0</td>
                        <td class="chart-cell">4.0-4.5</td>
                        <td class="chart-cell">4.5-5.0</td>
                        <td class="chart-cell">5.0-5.5</td>
                        <td class="chart-cell">6</td>
                      </tr>
                      <tr class="chart-row">
                        <td class="chart-cell row-label">الكتابة</td>
                        <td class="chart-cell">التعبير الأساسي</td>
                        <td class="chart-cell">فقرات بسيطة حول موضوع محدد</td>
                        <td class="chart-cell">القدرة الأساسية على الوصف والمقارنة</td>
                        <td class="chart-cell">الوصف والتحليل بطريقة منطقة</td>
                        <td class="chart-cell">مقال متقدم</td>
                      </tr>
                      <tr class="chart-row">
                        <td class="chart-cell row-label">القراءة</td>
                        <td class="chart-cell">تحديد الكلمات المفتاحية والقراءة السريعة</td>
                        <td class="chart-cell">فهم الفكرة الرئيسية وترتيب الإجابات</td>
                        <td class="chart-cell">تخمين الكلمات وتتبع الضمائر واستبعاد الخيارات</td>
                        <td class="chart-cell">تحليل الجمل وفهم الأفكار</td>
                        <td class="chart-cell">القراءة النقدية والتحليل المنطقي وتخمين الكلمات في السياق الصعب</td>
                      </tr>
                      <tr class="chart-row">
                        <td class="chart-cell row-label">القواعد</td>
                        <td class="chart-cell">★</td>
                        <td class="chart-cell">★★</td>
                        <td class="chart-cell">★★★</td>
                        <td class="chart-cell">★★★★</td>
                        <td class="chart-cell">★★★★★</td>
                      </tr>
                      <tr class="chart-row">
                        <td class="chart-cell row-label">حجم المفردات</td>
                        <td class="chart-cell">1500</td>
                        <td class="chart-cell">2500</td>
                        <td class="chart-cell">3500</td>
                        <td class="chart-cell">4500</td>
                        <td class="chart-cell">5500</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="report-footer">
        </div>
      </div>
      
      <!-- 底部导出按钮 -->
      <div class="bottom-export-section">
        <button class="export-btn" @click="exportAsImage" :disabled="isExporting">
          <span v-if="isExporting" class="loading-spinner">⏳</span>
          <span v-else>📸</span>
          {{ isExporting ? 'جاري التصدير...' : 'تصدير كصورة' }}
        </button>
      </div>
    </div>
  </div>
  </template>

<style scoped>
/* 只在教师页面的level按钮中应用高亮效果 */
:global(.level-btn .level-highlight) {
  font-weight: bold !important;
  color: #FCBB1F !important;
  background: none !important;
  padding: 0 !important;
  border: none !important;
  box-shadow: none !important;
  display: inline !important;
  margin-right: 0 !important;
}
.app {
  min-height: 100vh;
  background: #e8eef5;
  background-image: 
    radial-gradient(circle at 25% 25%, #FCBB1F33 0%, transparent 50%),
    radial-gradient(circle at 75% 75%, #3BC8D533 0%, transparent 50%),
    linear-gradient(45deg, transparent 40%, #FCBB1F15 50%, transparent 60%),
    linear-gradient(-45deg, transparent 40%, #3BC8D515 50%, transparent 60%);
  padding: 20px;
  font-family: 'IBM Plex Sans Arabic', 'Noto Sans Arabic', 'Arial', sans-serif;
  position: relative;
  color: #2c3e50;
  line-height: 1.6;
  font-size: 14px;
  overflow-x: hidden;
  overflow-y: auto;
  direction: rtl;
  text-align: right;
}

/* 导出时的特殊样式 */
.app.export-mode {
  overflow: visible !important;
  height: auto !important;
  min-height: auto !important;
  position: static !important;
}

.report-preview.export-ready {
  position: static !important;
  overflow: visible !important;
  height: auto !important;
  width: 100% !important;
  margin: 0 !important;
  transform: none !important;
  box-shadow: none !important;
  border-radius: 0 !important;
  /* 保持RTL布局但确保容器定位正确 */
  left: 0 !important;
  right: auto !important;
  top: 0 !important;
}

.back-section {
  margin-bottom: 30px;
  text-align: left;
}

.back-btn {
  background: #6c757d;
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 20px;
  font-size: 1.2rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(108, 117, 125, 0.2);
}

.back-btn:hover {
  background: #5a6268;
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(108, 117, 125, 0.3);
}





.container {
  max-width: 1200px;
  margin: 0 auto;
  position: relative;
  z-index: 2;
  min-height: calc(100vh - 40px);
  display: flex;
  flex-direction: column;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 20px;
  position: relative;
  flex-shrink: 0;
}

.header::after {
  content: '';
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 3px;
  background: linear-gradient(90deg, #3BC8D5, #FCBB1F);
  border-radius: 2px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.header h1 {
  font-size: 2.2rem;
  margin-bottom: 10px;
  text-shadow: 
    2px 2px 4px rgba(0,0,0,0.3),
    0 1px 2px rgba(255, 255, 255, 0.8);
  position: relative;
  color: #1a252f;
  line-height: 1.2;
  padding: 10px 0;
}

.header p {
  font-size: 1.1rem;
  opacity: 0.9;
}

.teacher-info {
  display: flex;
  justify-content: center;
  gap: 40px;
  flex-wrap: wrap;
  color: #2c3e50;
  margin-top: 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  backdrop-filter: blur(10px);
  direction: rtl;
}

.teacher-info .info-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 0;
}

.teacher-info .label {
  font-weight: bold;
  color: #1a252f;
  font-size: 1.1rem;
}

.teacher-info .value {
  color: #2c3e50;
  font-weight: 500;
  font-size: 1.1rem;
}

.refresh-btn {
  padding: 4px 8px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  background: rgba(59, 200, 213, 0.1);
  color: #3BC8D5;
  font-size: 0.9rem;
  transition: all 0.3s ease;
  margin-right: 8px;
}

.refresh-btn:hover:not(:disabled) {
  background: rgba(59, 200, 213, 0.2);
  transform: scale(1.05);
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.control-panel {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 15px;
  margin-bottom: 20px;
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.1),
    0 8px 16px rgba(0, 0, 0, 0.05),
    inset 0 1px 0 rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  display: flex;
  flex-direction: column;
  gap: 20px;
  position: relative;
  width: 100%;
  box-sizing: border-box;
}

.control-panel::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(59, 200, 213, 0.5), transparent);
}

.level-selector {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
  width: 100%;
  flex: 1;
  align-items: center;
}

.level-selector h3 {
  margin-bottom: 20px;
  color: #2c3e50;
  font-size: 1.4rem;
  text-align: center;
  line-height: 1.3;
  font-weight: 600;
}

.level-buttons {
  display: flex;
  flex-direction: column;
  gap: 12px;
  width: 100%;
  max-width: 600px;
  align-items: center;
}

.level-btn {
     display: flex;
     align-items: center;
     gap: 8px;
     padding: 8px 12px;
     margin-bottom: 8px;
     background: #ffffff;
     border-radius: 10px;
     border: 2px solid #3BC8D5;
     cursor: pointer;
     font-size: 13px;
     font-weight: 500;
     transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
     width: 80%;
     max-width: 500px;
     text-align: right;
     line-height: 1.2;
     white-space: normal;
     word-wrap: break-word;
     box-shadow: 
       0 4px 12px rgba(59, 200, 213, 0.15),
       0 2px 4px rgba(59, 200, 213, 0.1),
       inset 0 1px 0 rgba(255, 255, 255, 0.8);
     position: relative;
     overflow: hidden;
     min-height: 45px;
     justify-content: flex-end;
     flex-shrink: 0;
     margin-left: auto;
     margin-right: auto;
     box-sizing: border-box;
     direction: rtl;
   }

.level-btn:hover {
    background: #f8fdff;
    transform: translateY(-2px);
    box-shadow: 
      0 6px 20px rgba(252, 187, 31, 0.25),
      0 3px 8px rgba(59, 200, 213, 0.2),
      inset 0 1px 0 rgba(255, 255, 255, 0.9);
    border-color: #FCBB1F;
  }

  .level-btn.active {
    background: linear-gradient(135deg, #fff9e6 0%, #f0f9ff 100%);
    border: 3px solid #FCBB1F;
    box-shadow: 
      0 4px 15px rgba(252, 187, 31, 0.3),
      0 2px 6px rgba(59, 200, 213, 0.2),
      inset 0 2px 4px rgba(252, 187, 31, 0.1),
      inset 0 -1px 2px rgba(59, 200, 213, 0.1);
  }

  .level-number {
    font-size: 1rem;
    min-width: 30px;
  }

  .level-text {
     color: #2c3e50;
     font-weight: 500;
     flex: 1;
     line-height: 1.2;
     word-wrap: break-word;
     overflow-wrap: break-word;
     font-size: 0.9rem;
     text-align: right;
     max-width: 100%;
   }
   
   .level-text .level-highlight {
     color: #FCBB1F !important;
     font-weight: bold !important;
   }

  /* Level 1-5 字眼高亮样式 - 全局优先级 */
  .level-highlight {
    font-weight: bold !important;
    color: #FCBB1F !important;
    background: none !important;
    padding: 0 !important;
    border: none !important;
    box-shadow: none !important;
    display: inline !important;
    margin-right: 0 !important;
  }
  
  /* 确保在所有上下文中都生效 */
  * .level-highlight {
    font-weight: bold !important;
    color: #FCBB1F !important;
  }

.action-section {
   display: flex;
   gap: 15px;
   flex-wrap: wrap;
 }
 
 .crm-btn {
   background: #ffffff;
   color: #333;
   border: 2px solid #3BC8D5;
   padding: 15px 25px;
   border-radius: 25px;
   font-size: 1rem;
   font-weight: bold;
   cursor: pointer;
   transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
   box-shadow: 
     0 6px 16px rgba(59, 200, 213, 0.2),
     0 3px 6px rgba(59, 200, 213, 0.1),
     inset 0 1px 0 rgba(255, 255, 255, 0.8);
   position: relative;
   overflow: hidden;
 }
 
 .crm-btn:hover {
   background: #3BC8D5;
   color: white;
   transform: translateY(-2px);
   box-shadow: 
     0 6px 18px rgba(59, 200, 213, 0.3),
     0 3px 6px rgba(59, 200, 213, 0.2),
     inset 0 2px 4px rgba(255, 255, 255, 0.3);
 }
 
 .crm-btn:hover:not(:disabled) {
    transform: translateY(-3px);
    box-shadow: 0 8px 25px rgba(0,0,0,0.3);
  }
  
  .crm-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
  }
  
  .middle-submit-section {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 15px;
    margin: 20px 0;
    flex-shrink: 0;
  }
  
  .bottom-export-section {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 40px 30px;
    margin-top: 40px;
    background: transparent;
  }
  
  .submit-btn {
    background: #FCBB1F;
    color: white;
    border: none;
    padding: 20px 50px;
    border-radius: 25px;
    font-size: 1.4rem;
    font-weight: bold;
    cursor: pointer;
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 
      0 8px 20px rgba(252, 187, 31, 0.3),
      0 4px 8px rgba(252, 187, 31, 0.2),
      inset 0 2px 4px rgba(255, 255, 255, 0.3),
      inset 0 -2px 4px rgba(0, 0, 0, 0.1);
    min-width: 160px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    position: relative;
    overflow: hidden;
  }
  
  .submit-btn:hover {
    background: #ffcc4d;
    transform: translateY(-2px);
    box-shadow: 
      0 8px 20px rgba(252, 187, 31, 0.4),
      0 4px 8px rgba(252, 187, 31, 0.3),
      inset 0 2px 6px rgba(255, 255, 255, 0.4),
      inset 0 -2px 6px rgba(0, 0, 0, 0.1);
  }
  
  .export-btn {
    background: #3BC8D5;
    color: white;
    border: none;
    padding: 18px 45px;
    border-radius: 25px;
    font-size: 1.3rem;
    font-weight: bold;
    cursor: pointer;
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 
      0 8px 20px rgba(59, 200, 213, 0.3),
      0 4px 8px rgba(59, 200, 213, 0.2),
      inset 0 2px 4px rgba(255, 255, 255, 0.3),
      inset 0 -2px 4px rgba(0, 0, 0, 0.1);
    min-width: 160px;
    justify-content: center;
    position: relative;
    overflow: hidden;
  }
  
  .export-btn:hover:not(:disabled) {
    background: #4dd0e1;
    transform: translateY(-2px);
    box-shadow: 
      0 8px 20px rgba(59, 200, 213, 0.4),
      0 4px 8px rgba(59, 200, 213, 0.3),
      inset 0 2px 6px rgba(255, 255, 255, 0.4),
      inset 0 -2px 6px rgba(0, 0, 0, 0.1);
  }
  
  .export-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
  }
  
  .loading-spinner {
  display: inline-block;
}

.report-preview {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 50px;
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.1),
    0 8px 16px rgba(0, 0, 0, 0.05),
    inset 0 1px 0 rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  margin-bottom: 40px;
  position: relative;
  overflow: hidden;
  direction: rtl;
  text-align: right;
}

.report-preview::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(252, 187, 31, 0.5), transparent);
}

.report-header {
  text-align: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 3px solid #f0f0f0;
}

.report-header h2 {
  color: #1a252f;
  font-size: 2.5rem;
  margin-bottom: 30px;
  line-height: 1.4;
}

.report-info {
  display: flex;
  justify-content: center;
  gap: 50px;
  flex-wrap: wrap;
  color: #2c3e50;
  margin-bottom: 40px;
  direction: rtl;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 0;
}

.label {
  font-weight: bold;
  color: #666;
  font-size: 1.1rem;
}

.value {
   color: #333;
   font-weight: 500;
   font-size: 1.1rem;
 }
 
 .value.editable {
   cursor: pointer;
   padding: 4px 8px;
   border-radius: 4px;
   transition: background-color 0.2s ease;
 }
 
 .value.editable:hover {
   background-color: #f0f0f0;
 }
 
 .edit-name {
   display: flex;
   align-items: center;
   gap: 8px;
 }
 
 .name-input {
   padding: 8px 12px;
   border: 2px solid #e0e7ff;
   border-radius: 12px;
   font-size: 1rem;
   outline: none;
   min-width: 120px;
   background: #ffffff;
   transition: all 0.3s ease;
   box-shadow: 
     0 2px 8px rgba(59, 200, 213, 0.1),
     inset 0 1px 2px rgba(0, 0, 0, 0.05);
 }
 
 .name-input:focus {
  border-color: #3BC8D5;
  box-shadow: 
    0 4px 12px rgba(59, 200, 213, 0.2),
    0 0 0 3px rgba(59, 200, 213, 0.1),
    inset 0 1px 2px rgba(0, 0, 0, 0.05);
}
 
 .save-btn, .cancel-btn {
   padding: 4px 8px;
   border: none;
   border-radius: 4px;
   cursor: pointer;
   font-size: 0.9rem;
   font-weight: bold;
   transition: all 0.2s ease;
 }
 
 .save-btn {
   background-color: #4CAF50;
   color: white;
 }
 
 .save-btn:hover {
   background-color: #45a049;
 }
 
 .cancel-btn {
   background-color: #f44336;
   color: white;
 }
 
 .cancel-btn:hover {
   background-color: #da190b;
 }

.report-content {
  margin: 30px 0;
}

.content-section {
  margin-bottom: 50px;
}

.content-section h3 {
  color: #333;
  font-size: 1.8rem;
  margin-bottom: 30px;
  display: flex;
  align-items: center;
  gap: 8px;
  line-height: 1.4;
}

.content-box {
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(224, 231, 255, 0.6);
  border-right: 4px solid #3BC8D5;
  padding: 35px;
  border-radius: 16px;
  line-height: 1.6;
  box-shadow: 
    0 8px 20px rgba(59, 200, 213, 0.12),
    0 4px 8px rgba(59, 200, 213, 0.08),
    inset 0 1px 0 rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  position: relative;
  transition: all 0.3s ease;
  margin-bottom: 25px;
  direction: rtl;
  text-align: right;
}

.content-box:hover {
  box-shadow: 
    0 10px 25px rgba(59, 200, 213, 0.15),
    0 5px 10px rgba(59, 200, 213, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.9);
}

.content-box::before {
  content: '';
  position: absolute;
  top: 0;
  right: 4px;
  left: 0;
  height: 2px;
  background: linear-gradient(270deg, #3BC8D5, transparent, #FCBB1F);
  border-radius: 0;
}

.content-box p {
  margin: 0;
  color: #444;
  font-size: 1rem;
}

/* 学习项目样式 */
.learning-item {
  margin-bottom: 40px;
}

.learning-item h4 {
  color: #333;
  font-size: 1.4rem;
  margin-bottom: 25px;
  font-weight: bold;
  line-height: 1.4;
}

.word-list, .skill-list, .grammar-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.word-item, .skill-item, .grammar-item {
  display: flex;
  flex-direction: column;
  gap: 15px;
  padding: 15px 20px;
  background: #f0f8ff;
  border-radius: 8px;
  border-left: 3px solid #667eea;
  margin-bottom: 12px;
}

.word-number, .skill-number {
  font-size: 1rem;
  min-width: 30px;
}

.word-text, .skill-text, .grammar-text {
  color: #333;
  font-weight: 500;
  font-size: 1.1rem;
  line-height: 1.5;
}

.content-left {
  display: flex;
  align-items: center;
  gap: 15px;
}

.word-detail {
  color: #666;
  font-size: 0.9rem;
  font-weight: normal;
}

.skill-content, .grammar-content {
  width: 100%;
}

.skill-description, .grammar-description {
  color: #555;
  font-size: 0.95rem;
  line-height: 1.6;
  margin-top: 8px;
  padding-left: 45px;
  white-space: pre-line;
}

/* 评价模块样式 */
.evaluation-ratings {
  margin-bottom: 25px;
}

.evaluation-ratings h4 {
  color: #333;
  font-size: 1.1rem;
  margin-bottom: 15px;
  font-weight: bold;
}

.rating-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  gap: 10px;
  margin-bottom: 30px;
  overflow: hidden;
}

.rating-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 15px;
  background: #fff8dc;
  border-radius: 8px;
  border-right: 3px solid #ffa500;
  flex-wrap: nowrap;
  min-width: 0;
  direction: rtl;
}

.rating-label {
  font-weight: bold;
  color: #333;
  min-width: 50px;
  font-size: 1.1rem;
  white-space: nowrap;
  flex-shrink: 0;
}

.rating-stars {
  font-size: 1rem;
  white-space: nowrap;
  flex-shrink: 0;
}

:deep(.star) {
  display: inline-block;
  margin-right: 1px;
  font-size: 1.1rem;
  transition: color 0.2s ease;
  font-family: Arial, sans-serif;
}

:deep(.star.active) {
  color: #ff8c00 !important;
  text-shadow: 0 0 5px rgba(255, 140, 0, 0.7) !important;
  filter: brightness(1.2) !important;
}

:deep(.star.inactive) {
  color: #d3d3d3 !important;
  opacity: 0.6 !important;
  filter: grayscale(0.5) !important;
}

/* 可点击星星样式 */
.clickable-star {
  cursor: pointer;
  user-select: none;
  transition: all 0.2s ease;
}

.clickable-star:hover {
  transform: scale(1.2);
  filter: brightness(1.3) !important;
}

.clickable-star:active {
  transform: scale(0.9);
}

/* 鼠标悬停时的预览效果 */
.rating-stars:hover .clickable-star {
  opacity: 0.4;
}

.rating-stars:hover .clickable-star:hover,
.rating-stars:hover .clickable-star:hover ~ .clickable-star {
  opacity: 1;
}

.rating-stars .clickable-star:hover {
  color: #ff8c00 !important;
  text-shadow: 0 0 8px rgba(255, 140, 0, 0.8) !important;
}

.evaluation-section {
  margin-bottom: 35px;
}

.evaluation-section h4 {
  color: #333;
  font-size: 1.3rem;
  margin-bottom: 20px;
  font-weight: bold;
}

.evaluation-text {
  background: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  border-right: 3px solid #28a745;
  line-height: 1.8;
  color: #2c3e50;
  font-size: 1.05rem;
  direction: rtl;
  text-align: right;
}

.evaluation-line {
  margin-bottom: 15px;
  line-height: 1.6;
  color: #444;
  font-size: 1.05rem;
  position: relative;
  padding-right: 20px;
  direction: rtl;
  text-align: right;
}

.evaluation-line::before {
  content: '·';
  position: absolute;
  right: 0;
  top: 0;
  color: #28a745;
  font-weight: bold;
  font-size: 1.2rem;
}

.evaluation-line:last-child {
  margin-bottom: 0;
}

/* 等级展示样式 */
.level-display {
  text-align: center;
}

.current-level {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.level-badge {
  color: white;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 13px;
  font-weight: bold;
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
  line-height: 1.4;
  max-width: 100%;
  word-wrap: break-word;
  margin: 15px 0;
  text-align: left;
  align-self: flex-start;
}

.level-description {
  color: #666;
  font-style: italic;
  margin: 0;
}

.level-chart {
  background: #fff;
  overflow: hidden;
  margin: 20px 0;
  font-size: 14px;
  direction: rtl;
}

.chart-table {
  width: 100%;
  border-collapse: collapse;
}

.chart-header {
  height: 100px;
}

.chart-header th {
  color: white;
  padding: 12px 8px;
  text-align: center;
  font-size: 14px;
  font-weight: bold;
  line-height: 1.4;
  position: relative;
}

.chart-row td {
  border: 1px solid #e9ecef;
  padding: 12px 8px;
  text-align: center;
  font-size: 13px;
  line-height: 1.4;
}

.chart-row:nth-child(even) {
  background: #f8f9fa;
}

.row-label-header {
      background: linear-gradient(179deg, #fff8dc 0%, #fff8dc 100%);
    color: #333 !important;
    background-size: 100% 40px;
  width: 120px;
}

.row-label {
  background: #fff8dc;
  font-weight: bold;
  color: #333;
  border-left: 2px solid #ffa500;
  width: 120px;
}

/* 阶梯效果 - 使用背景图高度实现 */
.chart-header {
  height: 100px;
}

.chart-header th {
  vertical-align: bottom;
  position: relative;
  padding: 8px;
  background-repeat: no-repeat;
  background-position: bottom;
}

.level1-header {
  background: linear-gradient(135deg, #3BC8D5 0%, #2196F3 100%);
  background-size: 100% 40px;
  background-position: bottom;
  color: white;
}

.level2-header {
  background: linear-gradient(135deg, #3BC8D5 0%, #2196F3 100%);
  background-size: 100% 55px;
  background-position: bottom;
  color: white;
}

.level3-header {
  background: linear-gradient(135deg, #3BC8D5 0%, #2196F3 100%);
  background-size: 100% 70px;
  background-position: bottom;
  color: white;
}

.level4-header {
  background: linear-gradient(135deg, #3BC8D5 0%, #2196F3 100%);
  background-size: 100% 85px;
  background-position: bottom;
  color: white;
}

.level5-header {
  background: linear-gradient(135deg, #3BC8D5 0%, #2196F3 100%);
  background-size: 100% 100px;
  background-position: bottom;
  color: white;
}

.level6-header {
  background: linear-gradient(135deg, #795548 0%, #5d4037 100%);
  background-size: 100% 100px;
  background-position: bottom;
  color: white;
}

.row-label-header {
  color: #333 !important;
}

.chart-cell:not(.row-label):not(.header-cell) {
  transition: background-color 0.2s ease;
}

.chart-cell:not(.row-label):not(.header-cell):hover {
  background: #e3f2fd;
}

/* 维度文案显示样式 */
.dimension-texts {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.dimension-text-item {
  margin-bottom: 10px;
}

.dimension-text-content {
  background: #f8f9fa;
  border: 1px solid #e9ecef;
  border-left: 4px solid #007bff;
  border-radius: 8px;
  padding: 15px 20px;
  transition: all 0.3s ease;
}

.dimension-text-content:hover {
  background: #f1f3f4;
  border-left-color: #0056b3;
  box-shadow: 0 2px 8px rgba(0, 123, 255, 0.1);
}

.dimension-text {
  color: #333;
  font-size: 1.05rem;
  line-height: 1.6;
  margin: 0;
  padding: 5px;
  border-radius: 4px;
  transition: all 0.2s ease;
  min-height: 24px;
  cursor: text;
}

.dimension-text:hover {
  background: rgba(0, 123, 255, 0.05);
}

.dimension-text:focus {
  outline: none;
  background: #fff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.2);
}

.dimension-text.editing {
  background: #fff;
  border: 1px solid #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.1);
}

.dimension-text .placeholder-text {
  color: #999;
  font-style: italic;
}

.report-footer {
  text-align: center;
  margin-top: 50px;
  padding-top: 25px;
  border-top: 2px solid #f0f0f0;
  color: #888;
  font-style: italic;
  font-size: 1rem;
  line-height: 1.6;
}

@media (max-width: 768px) {
  .control-panel {
    flex-direction: column;
    align-items: stretch;
  }
  
  .level-buttons {
    justify-content: center;
  }
  
  .level-btn {
    min-width: 100%;
    max-width: 100%;
    margin: 0 0 8px 0;
    font-size: 12px;
    padding: 10px 12px;
  }
  
  .level-selector {
    justify-content: center;
  }
  
  .report-info {
    flex-direction: column;
    gap: 15px;
  }
  
  .report-preview {
    padding: 20px;
  }
  
  .header h1 {
    font-size: 2rem;
  }
  
  .rating-grid {
    grid-template-columns: 1fr;
  }
  
  .word-item, .skill-item, .grammar-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 5px;
  }
  
  .word-number, .skill-number {
    min-width: auto;
  }
  
  .level-placeholder {
    padding: 20px;
    min-height: 80px;
    font-size: 1rem;
  }

/* 内联编辑样式 */
.editable-content {
  min-height: 120px;
  padding: 15px;
  border: 2px solid transparent;
  border-radius: 8px;
  cursor: text;
  transition: all 0.3s ease;
  background: #fafafa;
  position: relative;
}

.editable-content:hover {
  background: #f0f8ff;
  border-color: #e3f2fd;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.editable-content:focus {
  outline: none;
  background: white;
  border-color: #4CAF50;
  box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.1);
}

.editable-content.editing {
  background: white;
  border-color: #4CAF50;
  box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.1);
}

.editable-content p {
  margin: 0 0 12px 0;
  line-height: 1.6;
}

.editable-content p:last-child {
  margin-bottom: 0;
}

.placeholder-text {
  color: #999;
  font-style: italic;
  margin: 0;
}

.editable-content:hover .placeholder-text {
  color: #666;
}

.editable-content:focus .placeholder-text {
  display: none;
}
  
  .action-section {
    flex-direction: column;
  }
  
  .middle-submit-section {
    padding: 15px;
  }
  
  .bottom-export-section {
    padding: 20px 15px;
  }
  
  .submit-btn, .export-btn {
    width: 90%;
    max-width: 300px;
    margin: 0;
    text-align: center;
  }
}

/* 阿拉伯语字体样式 - 恢复最初的简单配置 */
* {
  font-family: 'IBM Plex Sans Arabic', 'Noto Sans Arabic', 'Arial', sans-serif !important;
}

/* 确保阿拉伯语文本在导出时不会意外换行 */
.export-ready .content-section h3,
.export-ready .content-section p,
.export-ready .content-section div {
  line-height: 1.6 !important;
  word-spacing: normal !important;
  letter-spacing: normal !important;
}

/* 防止表格和列表在导出时布局错乱 */
.export-ready table,
.export-ready .skills-table {
  table-layout: fixed !important;
  width: 100% !important;
}

.export-ready td,
.export-ready th {
  word-wrap: break-word !important;
  overflow-wrap: break-word !important;
}

/* 阿拉伯语文本专门优化 */
.export-ready [dir="rtl"],
.export-ready .arabic-text {
  direction: rtl !important;
  text-align: right !important;
  unicode-bidi: embed !important;
  word-break: keep-all !important;
  overflow-wrap: normal !important;
  hyphens: none !important;
}

/* 防止阿拉伯语单词被意外拆分 */
.export-ready .content-section,
.export-ready .report-header,
.export-ready .student-info {
  word-break: keep-all !important;
  overflow-wrap: normal !important;
  white-space: normal !important;
}

/* 确保表格中的阿拉伯语文本正确显示 */
.export-ready .skills-table td,
.export-ready .skills-table th {
  direction: rtl !important;
  text-align: right !important;
  word-break: keep-all !important;
  white-space: nowrap !important;
  overflow: hidden !important;
  text-overflow: ellipsis !important;
}
</style>
