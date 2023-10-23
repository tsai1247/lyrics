<template>
  <div>
    <v-row>
      <v-col cols="2">
        <!-- 曲名 -->
        <v-autocomplete
          label="曲名"
          v-model="playing"
          :items="lyrics"
          item-title="name"
          return-object
        ></v-autocomplete>
      </v-col>

      <v-col cols="2">
        <!-- 曲名 -->
        <v-checkbox
          label="顯示假名"
          v-model="showHirakana"
        >
        </v-checkbox>
      </v-col>

      <v-col cols="12">
        <div
          class="playing-lyrics"
          v-if="playing"
        >
          <div
            v-if="playing.format === 'ruby'"
            v-html="formatLyric(playing.lyrics)"
          >
          </div>
          <div
            v-else-if="playing.format === 'docs'"
            v-html="formatLyric(docsLyrics)"
          >
          </div>
          <div v-else>{{ notSupport }}</div>
        </div>
      </v-col>
    </v-row>
  </div>

</template>

<script setup>
import { ref, computed } from 'vue';
import lyrics from '@/assets/lyrics.json'

const playing = ref(lyrics[0]);
const notSupport = computed(() => {
  return `未支援的格式: ${playing.value.lyrics}`;
});

const docsLyrics = computed( () => {
  const tmpList = [];
  let start = 0;
  for (let i = 0; i < playing.value.lyrics.length; i++) {
    if(playing.value.lyrics[i] === ')' || playing.value.lyrics[i] === '\n') {
      tmpList.push(playing.value.lyrics.substring(start, i+1));
      start = i + 1;
    }
  }
  if(start < playing.value.lyrics.length) {
    tmpList.push(playing.value.lyrics.substring(start, playing.value.lyrics.length));
  }

  const lyricslist = [];
  tmpList.forEach(item => {
    const hirakana_start = item.indexOf('(') === -1 ? item.length : item.indexOf('(');
    const hirakana_end = item.indexOf(')');
    if(hirakana_end === -1) {
      lyricslist.push({
        'kanji': item.substring(0, item.length),
      });
    }
    else {
      let i = hirakana_start;
      for (; i >= 0; i--) {
        const target = item[i].charCodeAt(0);
        if (target >= 12353 && target <= 12543 ||
        item[i] >= 'a' && item[i] <= 'z' ||
        item[i] >= 'A' && item[i] <= 'Z' ||
        item[i] >= '0' && item[i] <= '9') {

          lyricslist.push({
            'kanji': item.substring(0, i + 1),
          });


          break;
        }
      }
      lyricslist.push({
            'kanji': item.substring(i + 1, hirakana_start),
            'hirakana': item.substring(hirakana_start + 1, hirakana_end)
          });
    }

  });

  let ret = '<div>';
  lyricslist.forEach(item => {
    const kanji = item.kanji.replace('\n', '');
    if(item.hirakana) {
      ret += `<ruby><rb>${kanji}</rb><rt>${item.hirakana}</rt></ruby>`;
    }
    else {
      ret += `<ruby>${kanji}</ruby>`;
    }
    if(item.kanji.indexOf('\n') > 0) {
      ret += '</div><div>';
    }
  })
  ret += '</div>';

  return ret;
});

const showHirakana = ref(true);
function formatLyric (lyric) {
  if (showHirakana.value) {
    return lyric;
  }

  let ret = lyric;
  while(ret.indexOf("<rt>") >= 0) {
    const start = ret.indexOf("<rt>");
    const end = ret.indexOf("</rt>");
    ret = ret.slice(0, start) + ret.slice(end + 5);
  }

  return ret;
}
</script>

<style scoped>
  .playing-lyrics {
    font-size: 24px;
    line-height: 60px;
    margin-left: 15px;
  }
</style>
