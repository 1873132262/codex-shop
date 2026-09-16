<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'
import ReservationModal from './components/ReservationModal.vue'

const navOpen = ref(false)
const reservationOpen = ref(false)
const isScrolled = ref(false)
let revealObserver = null

const signatureDishes = [
  {
    index: '01',
    name: '龙井虾仁',
    price: '¥ 100',
    description: '明前龙井轻焙出香，搭配手剥河虾仁，清鲜回甘，是春日餐桌上最轻盈的一味。',
    tag: '春季限定',
    image: 'https://images.unsplash.com/photo-1547592180-85f173990554?auto=format&fit=crop&w=900&q=82',
    alt: '清鲜爽嫩的龙井虾仁'
  },
  {
    index: '02',
    name: '黑松露和牛',
    price: '¥ 268',
    description: '和牛以文火慢煨至酥软，黑松露的木质香气与醇厚肉汁层层交叠，丰润不腻。',
    tag: '主厨推荐',
    image: 'https://images.unsplash.com/photo-1544025162-d76694265947?auto=format&fit=crop&w=900&q=82',
    alt: '慢火烹制的黑松露和牛'
  },
  {
    index: '03',
    name: '金汤花胶黄鱼',
    price: '¥ 198',
    description: '黄鱼鲜嫩，花胶软糯，以金黄清汤托住海味。入口温润，适合宴席中段慢慢品味。',
    tag: '宴客首选',
    image: 'https://images.unsplash.com/photo-1525755662778-989d0524087e?auto=format&fit=crop&w=1000&q=82',
    alt: '金汤花胶黄鱼'
  }
]

const banquetPackages = [
  {
    kicker: 'FAMILY GATHERING',
    name: '合家欢宴',
    description: '适合家庭团聚与长辈寿宴，八道式菜单兼顾经典口味与时令新意。',
    features: ['6–8 位', '8 道菜', '独立雅间'],
    price: '¥ 1,288 / 桌',
    image: 'https://images.unsplash.com/photo-1552566626-52f8b828add9?auto=format&fit=crop&w=1100&q=82',
    alt: '家庭聚餐使用的圆桌雅间'
  },
  {
    kicker: 'BUSINESS DINING',
    name: '雅集商务宴',
    description: '兼顾私密与体面，十道式臻选菜单、专属茶艺与分餐服务可按需安排。',
    features: ['10–12 位', '10 道菜', '专属服务'],
    price: '¥ 2,688 / 桌',
    image: 'https://images.unsplash.com/photo-1414235077428-338989a2e8c0?auto=format&fit=crop&w=1100&q=82',
    alt: '适合商务宴请的精致餐厅包间'
  }
]

const galleryItems = [
  {
    title: '主用餐区',
    detail: '自然光 · 木与石',
    image: 'https://images.unsplash.com/photo-1559339352-11d035aa65de?auto=format&fit=crop&w=1200&q=82',
    alt: '悦膳宽敞明亮的主用餐区'
  },
  {
    title: '细节陈设',
    detail: '一席一器',
    image: 'https://images.unsplash.com/photo-1516211697506-8360dbcfe9a4?auto=format&fit=crop&w=800&q=82',
    alt: '餐厅内细致的餐桌陈设'
  },
  {
    title: '开放料理',
    detail: '看得见的新鲜',
    image: 'https://images.unsplash.com/photo-1515003197210-e0cd71810b5f?auto=format&fit=crop&w=800&q=82',
    alt: '烹饪完成并精心摆盘的时令菜品'
  },
  {
    title: '独立雅间',
    detail: '6–12 位',
    image: 'https://images.unsplash.com/photo-1541544741938-0af808871cc0?auto=format&fit=crop&w=800&q=82',
    alt: '适合宴请的独立用餐雅间'
  },
  {
    title: '宴客圆桌',
    detail: '从容落座',
    image: 'https://images.unsplash.com/photo-1504674900247-0877df9cc836?auto=format&fit=crop&w=800&q=82',
    alt: '餐桌上丰富的时令菜品'
  }
]

const reviews = [
  {
    text: '“带父母来吃寿宴，菜不重口却很有层次。服务分寸拿捏得刚好，长辈很满意。”',
    initial: '林',
    name: '林女士',
    occasion: '家庭寿宴 · 8 人'
  },
  {
    text: '“商务晚宴选在悦膳很稳妥，包间安静，上菜节奏和菜品讲解都让人省心。”',
    initial: '周',
    name: '周先生',
    occasion: '商务宴请 · 10 人'
  },
  {
    text: '“没有刻意堆叠名贵食材，但每道菜都很清楚、很新鲜。金汤黄鱼尤其让人惦记。”',
    initial: '许',
    name: '许先生',
    occasion: '朋友小聚 · 4 人'
  }
]

function handleScroll() {
  isScrolled.value = window.scrollY > 18
}

function openReservation() {
  navOpen.value = false
  reservationOpen.value = true
}

function closeReservation() {
  reservationOpen.value = false
}

function handleImageError(event) {
  event.currentTarget.classList.add('is-broken')
}

function handleDocumentClick(event) {
  const header = document.getElementById('site-header')
  if (navOpen.value && header && !header.contains(event.target)) {
    navOpen.value = false
  }
}

function handleKeydown(event) {
  if (event.key === 'Escape' && navOpen.value) {
    navOpen.value = false
  }
}

onMounted(() => {
  handleScroll()
  window.addEventListener('scroll', handleScroll, { passive: true })
  document.addEventListener('click', handleDocumentClick)
  document.addEventListener('keydown', handleKeydown)

  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  const elements = document.querySelectorAll('.reveal')

  if (reduceMotion || !('IntersectionObserver' in window)) {
    elements.forEach((element) => element.classList.add('is-visible'))
    return
  }

  revealObserver = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-visible')
        revealObserver.unobserve(entry.target)
      }
    })
  }, {
    threshold: 0.12,
    rootMargin: '0px 0px -8% 0px'
  })

  elements.forEach((element) => revealObserver.observe(element))
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll)
  document.removeEventListener('click', handleDocumentClick)
  document.removeEventListener('keydown', handleKeydown)
  if (revealObserver) revealObserver.disconnect()
})
</script>

<template>
  <a class="skip-link" href="#main-content">跳到主要内容</a>

  <header id="site-header" class="site-header" :class="{ 'is-scrolled': isScrolled, 'nav-is-open': navOpen }">
    <div class="container nav-shell">
      <a class="brand" href="#home" aria-label="悦膳首页" @click="navOpen = false">
        <span class="brand-mark" aria-hidden="true">悦</span>
        <span class="brand-copy">
          <span class="brand-name">悦膳</span>
          <span class="brand-en">YUE SHAN</span>
        </span>
      </a>

      <nav id="site-nav" class="site-nav" :class="{ 'is-open': navOpen }" aria-label="主导航">
        <a class="nav-link" href="#signature" aria-current="page" @click="navOpen = false">招牌味道</a>
        <a class="nav-link" href="#banquet" @click="navOpen = false">宴席套餐</a>
        <a class="nav-link" href="#story" @click="navOpen = false">悦膳故事</a>
        <a class="nav-link" href="#space" @click="navOpen = false">餐厅环境</a>
        <a class="nav-link" href="#reviews" @click="navOpen = false">食客口碑</a>
      </nav>

      <div class="nav-actions">
        <a class="phone-link" href="tel:02162888899">021-6288 8899</a>
        <button class="btn btn--primary nav-reserve" type="button" @click="openReservation">立即订座</button>
        <button
          id="mobile-toggle"
          class="mobile-toggle"
          type="button"
          :aria-expanded="navOpen"
          aria-controls="site-nav"
          :aria-label="navOpen ? '关闭导航菜单' : '打开导航菜单'"
          @click="navOpen = !navOpen"
        >
          <span></span>
          <span></span>
          <span></span>
        </button>
      </div>
    </div>
  </header>

  <main id="main-content">
    <section id="home" class="hero">
      <div class="container hero-grid">
        <div class="hero-copy reveal">
          <p class="eyebrow">Modern Chinese Dining</p>
          <h1 class="hero-title">一席东方味<br><em>宴四方来客</em></h1>
          <p class="hero-intro">顺时而食，以江南风物入席。我们以克制的调味呈现食材本真，也为每一次团聚留出从容与体面。</p>
          <div class="hero-actions">
            <button class="btn btn--primary" type="button" @click="openReservation">立即订座 <span class="arrow" aria-hidden="true">→</span></button>
            <a class="btn btn--ghost" href="#signature">浏览菜单</a>
          </div>
          <div class="hero-meta" aria-label="餐厅概览">
            <div>
              <span>营业时间</span>
              <strong>11:00 — 21:30</strong>
            </div>
            <div>
              <span>餐厅地址</span>
              <strong>静安区 · 愚园路 618 号</strong>
            </div>
            <div>
              <span>雅间服务</span>
              <strong>6 间独立包房</strong>
            </div>
          </div>
        </div>

        <div class="hero-visual reveal reveal-delay-1">
          <figure class="hero-main">
            <img
              src="https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?auto=format&amp;fit=crop&amp;w=1100&amp;q=84"
              alt="悦膳餐厅温暖雅致的用餐空间"
              fetchpriority="high"
              @error="handleImageError"
            >
          </figure>
          <div class="hero-seal" aria-hidden="true">时令 · 本味</div>
          <figure class="hero-small">
            <img
              src="https://images.unsplash.com/photo-1563245372-f21724e3856d?auto=format&amp;fit=crop&amp;w=700&amp;q=82"
              alt="手工制作的中式点心"
              loading="lazy"
              @error="handleImageError"
            >
          </figure>
          <aside class="hero-note">
            <span>今日推荐</span>
            <strong>糟香六月黄</strong>
          </aside>
        </div>
      </div>
    </section>

    <div class="brand-strip" aria-hidden="true">
      <div class="container brand-strip-inner">
        <span>顺时而食</span>
        <i></i>
        <span>江南本味</span>
        <i></i>
        <span>一席款待</span>
        <i></i>
        <span>宾主尽欢</span>
      </div>
    </div>

    <section id="signature" class="section signature">
      <div class="container">
        <div class="section-heading reveal">
          <div>
            <p class="eyebrow">Signature Dishes</p>
            <h2 class="section-title">一席一味，<em>各有讲究</em></h2>
          </div>
          <p>不追逐浓烈，只让鲜、香、嫩、润在恰当的时候相遇。每一道招牌菜，都从时令与食材本身出发。</p>
        </div>

        <div class="dish-grid">
          <article
            v-for="(dish, index) in signatureDishes"
            :key="dish.name"
            class="dish-card reveal"
            :class="'reveal-delay-' + Math.min(index, 2)"
          >
            <div class="dish-image">
              <span class="dish-index">{{ dish.index }}</span>
              <img :src="dish.image" :alt="dish.alt" loading="lazy" @error="handleImageError">
            </div>
            <div class="dish-body">
              <div class="dish-topline">
                <h3 class="dish-title">{{ dish.name }}</h3>
                <span class="dish-price">{{ dish.price }}</span>
              </div>
              <p class="dish-description">{{ dish.description }}</p>
              <span class="dish-tag">{{ dish.tag }}</span>
            </div>
          </article>
        </div>
      </div>
    </section>

    <section id="banquet" class="section banquet">
      <div class="container">
        <div class="section-heading reveal">
          <div>
            <p class="eyebrow">Private Banquet</p>
            <h2 class="section-title">宴有分寸，<em>款待有形</em></h2>
          </div>
          <p>从菜单搭配、上菜节奏到席间服务，为家宴与商务宴请提供恰到好处的安排。</p>
        </div>

        <div class="banquet-grid">
          <article
            v-for="(item, index) in banquetPackages"
            :key="item.name"
            class="banquet-card reveal"
            :class="'reveal-delay-' + Math.min(index, 2)"
          >
            <div class="banquet-bg">
              <img :src="item.image" :alt="item.alt" loading="lazy" @error="handleImageError">
            </div>
            <div class="banquet-content">
              <span class="banquet-kicker">{{ item.kicker }}</span>
              <h3 class="banquet-title">{{ item.name }}</h3>
              <p class="banquet-description">{{ item.description }}</p>
              <div class="banquet-features">
                <span v-for="feature in item.features" :key="feature">{{ feature }}</span>
              </div>
              <div class="banquet-bottom">
                <div class="banquet-price">
                  <span>参考价</span>
                  <strong>{{ item.price }}</strong>
                </div>
                <button class="btn btn--light" type="button" @click="openReservation">预订此套餐</button>
              </div>
            </div>
          </article>
        </div>
      </div>
    </section>

    <section id="story" class="section story">
      <div class="container story-grid">
        <div class="story-visual reveal">
          <figure class="story-photo">
            <img
              src="https://images.unsplash.com/photo-1504674900247-0877df9cc836?auto=format&amp;fit=crop&amp;w=1000&amp;q=84"
              alt="摆放于餐桌上的时令中式菜肴"
              loading="lazy"
              @error="handleImageError"
            >
          </figure>
          <div class="story-quote">
            <strong>“不时不食，<br>不鲜不烹。”</strong>
            <span>悦膳 · 料理理念</span>
          </div>
        </div>

        <div class="story-copy reveal reveal-delay-1">
          <p class="eyebrow">Our Story</p>
          <h2 class="section-title">让食物，<em>顺着时间生长</em></h2>
          <p>悦膳始于 2014 年，从一间只有八张桌子的小馆出发，坚持跟着二十四节气寻找食材。春笋、夏藕、秋蟹、冬菌，都被认真记在厨房的季节表上。</p>
          <p>我们相信，现代中餐不是推翻传统，而是让老味道拥有更清晰、轻盈的表达。少一分修饰，多一分食材本来的温度。</p>
          <div class="story-stats">
            <div class="story-stat">
              <strong>12 年</strong>
              <span>专注现代中餐</span>
            </div>
            <div class="story-stat">
              <strong>28 道</strong>
              <span>时令菜单轮换</span>
            </div>
            <div class="story-stat">
              <strong>6 间</strong>
              <span>独立宴客雅间</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="space" class="section space">
      <div class="container">
        <div class="section-heading reveal">
          <div>
            <p class="eyebrow">The Space</p>
            <h2 class="section-title">留白之间，<em>自有温度</em></h2>
          </div>
          <p>木、石、纸与暖光构成安静的空间底色。大厅适合轻松相聚，独立雅间则为重要时刻保留私密。</p>
        </div>

        <div class="space-grid">
          <figure
            v-for="(item, index) in galleryItems"
            :key="item.title"
            class="gallery-item reveal"
            :class="'reveal-delay-' + Math.min(index % 3, 2)"
          >
            <img :src="item.image" :alt="item.alt" loading="lazy" @error="handleImageError">
            <figcaption class="gallery-caption">
              <span>{{ item.title }}</span>
              <span>{{ item.detail }}</span>
            </figcaption>
          </figure>
        </div>
      </div>
    </section>

    <section id="reviews" class="section reviews">
      <div class="container">
        <div class="section-heading reveal">
          <div>
            <p class="eyebrow">Guest Words</p>
            <h2 class="section-title">席散之后，<em>还有余味</em></h2>
          </div>
          <p>一餐饭真正留下的，不只是味道，还有被妥帖照顾的时刻。</p>
        </div>

        <div class="reviews-grid">
          <article
            v-for="(review, index) in reviews"
            :key="review.name"
            class="review-card reveal"
            :class="'reveal-delay-' + Math.min(index, 2)"
          >
            <div class="review-stars" aria-label="5 星评价">★★★★★</div>
            <blockquote class="review-text">{{ review.text }}</blockquote>
            <div class="review-author">
              <span class="avatar" aria-hidden="true">{{ review.initial }}</span>
              <span>
                <strong>{{ review.name }}</strong>
                <span>{{ review.occasion }}</span>
              </span>
            </div>
          </article>
        </div>
      </div>
    </section>

    <section id="reserve" class="reserve-cta">
      <div class="container reserve-cta-inner">
        <div class="reveal">
          <p class="eyebrow">Reservation</p>
          <h2>把重要的相聚，交给一席好饭。</h2>
          <p>建议提前 1–3 天预订。如需指定雅间、安排菜单或有忌口需求，请在预订时告诉我们。</p>
        </div>
        <div class="reserve-actions reveal reveal-delay-1">
          <button class="btn btn--light" type="button" @click="openReservation">在线订座 <span class="arrow" aria-hidden="true">→</span></button>
          <a class="btn btn--outline-light" href="tel:02162888899">电话咨询</a>
          <span class="reserve-contact">每日 10:30 — 21:00</span>
        </div>
      </div>
    </section>
  </main>

  <footer class="site-footer">
    <div class="container">
      <div class="footer-grid">
        <div class="footer-brand">
          <a class="brand" href="#home" aria-label="悦膳首页">
            <span class="brand-mark" aria-hidden="true">悦</span>
            <span class="brand-copy">
              <span class="brand-name">悦膳</span>
              <span class="brand-en">YUE SHAN</span>
            </span>
          </a>
          <p class="footer-about">以江南风物为灵感，以现代手法演绎中餐本味。为家宴、商务与每一次值得纪念的相聚，认真准备一席。</p>
        </div>

        <div class="footer-column">
          <h3>快速导航</h3>
          <a href="#signature">招牌味道</a>
          <a href="#banquet">宴席套餐</a>
          <a href="#story">悦膳故事</a>
          <a href="#space">餐厅环境</a>
        </div>

        <div class="footer-column">
          <h3>营业信息</h3>
          <p>午市 11:00 — 14:00</p>
          <p>晚市 17:00 — 21:30</p>
          <p>全年无休</p>
        </div>

        <div class="footer-column">
          <h3>联系我们</h3>
          <a href="tel:02162888899">021-6288 8899</a>
          <a href="mailto:hello@yueshan.example">hello@yueshan.example</a>
          <p>上海市静安区愚园路 618 号</p>
        </div>
      </div>

      <div class="footer-bottom">
        <span>© {{ new Date().getFullYear() }} 悦膳现代中餐厅 · 示例网站</span>
        <span><a href="#home">返回顶部 ↑</a></span>
      </div>
    </div>
  </footer>

  <ReservationModal :open="reservationOpen" @close="closeReservation" />
</template>
