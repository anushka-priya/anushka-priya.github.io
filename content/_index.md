---
title: "Anushka Priya"
layout: "landing_page"
---
Hi! I'm Anushka, an undergraduate student at Manipal Institute of Technology majoring in Electronics and Communication Engineering.
I'm currently figuring out my interests & navigating life- one commit at a time.
Feel free to explore my work, or reach out via email!

<div style="display: flex; justify-content: center; gap: 40px; margin-top: 60px;">

  <a href="/projects" style="text-decoration:none;">
    <div class="flip-card" style="width:200px; height:200px;">
      <div class="flip-inner">
        <div class="flip-front"><img src="/images/sticker.png" style="width:200px; height:200px; object-fit:contain;"></div>
        <div class="flip-back">projects</div>
      </div>
    </div>
  </a>

  <a href="/blog" style="text-decoration:none;">
    <div class="flip-card" style="width:200px; height:200px;">
      <div class="flip-inner">
        <div class="flip-front"><img src="/images/blog.png" style="width:200px; height:200px; object-fit:contain;"></div>
        <div class="flip-back">blog</div>
      </div>
    </div>
  </a>

  <a href="/publications" style="text-decoration:none;">
    <div class="flip-card" style="width:200px; height:200px;">
      <div class="flip-inner">
        <div class="flip-front"><img src="/images/cite.png" style="width:200px; height:200px; object-fit:contain;"></div>
        <div class="flip-back">publications</div>
      </div>
    </div>
  </a>

</div>

<style>
.flip-card { perspective: 1000px; cursor: pointer; }
.flip-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
.flip-card:hover .flip-inner { transform: rotateY(180deg); }
.flip-front, .flip-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; border-radius: 16px; display: flex; align-items: center; justify-content: center; }
.flip-back { background: #ffb6c1; color: #2d1f2e; font-size: 22px; font-family: Georgia, serif; font-style: italic; transform: rotateY(180deg); }
@media (max-width: 600px) {
  .flip-card { width: 120px !important; height: 120px !important; }
  .flip-card img { width: 120px !important; height: 120px !important; }
  .flip-back { font-size: 16px !important; }
}
</style>