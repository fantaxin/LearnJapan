{% include base.html %}
<div>
模式:
- <a class="toggle-mode" data-column="0|2|3|4|5">普通浏览</a>
- <a class="toggle-mode" data-column="2|3|5|6">看中文忆日文</a>
- <audio style="display:none" id="jpAudio" controls></audio>
</div>

| 假名    | 汉字 | 词性 | 解释 | 单词 | 课 | 序号 |
| ----    | ---- | ---- | ---- | ---- | -- | --   |
| loading |      |      |      |      |    |      |
| ====    | ==== | ==== | ==== | ==== | == | ==   |
{:.display.table.table-striped.table-bordered width="100%"}

<script>
function speak(lesson, wordIndex) {
  lesson = lesson.substr(2,2);
  wordIndex = String(+wordIndex-1).padStart(2, "0");
  var audio = $("#jpAudio")[0];
  audio.src = "{{ basepath }}/audio/book01-lesson"+lesson+"-"+wordIndex+".mp3";
  audio.play();
};
$(document).ready(function() {
  function inittable() {
    table.ajax.url('{{ basepath }}/words.json' ).load(function (){
      table.column(0).nodes().to$().addClass('japan');
      table.column(1).nodes().to$().addClass('japan');
      table.column(4).nodes().to$().addClass('japan');
      table
        .order( [5, 'asc'], [6, 'asc'] )
        .draw();
    }, false);
    table.on('xhr.dt', function ( e, settings, json, xhr ) {
      json.data.forEach(function(part, index, arr) {
        var content = arr[index][1];
        if(content.length == 0){
          content = arr[index][0].split("@")[0];
        }
        arr[index][0]=japanruby(arr[index][0]);
        arr[index][4]=japanruby(arr[index][4]);
        //arr[index][1] = '<a href="http://kanji.jitenon.jp/cat/search.php?getdata=' + content + '" target="_blank">' + content + '</a>';
        var lesson = arr[index][5];
        var wordIndex = arr[index][6];
        arr[index][1] = '<a onclick="speak(\''+lesson+'\',\''+wordIndex+'\')">' + content + '</a>';
      });
    });
  }
  setTimeout(inittable, 300);
  $('a.toggle-mode').on('click', function(e) {
    e.preventDefault();
    table.columns().visible(false);
    $.each($(this).attr('data-column').split(/\|/), function (i, cnum) {
        var column = table.column(cnum);
        column.visible(true);
    })
  });
  $('button.toggle-start').on('click', function(e) {
    e.preventDefault();
    var quizdata = table.rows({filter: 'applied'}).data()
      .map(function(p) { return { kana: p[0], kanji: p[1], explain: p[3], display: p[4], pos: p[2], rid: p[5] + '|' + p[6]}})
      .map(function(p) {
        p.kana = japanruby(p.kana);
        p.purekana = p.kana.replace(/[^\u3040-\u309f\u30a0-\u30ff]/g, "");
        p.display = japanruby(p.display);
        var desc = "<span class='japan'>" + (p.kanji == "&nbsp;" ? p.kana : p.display + "<br />" + p.kana) + "</span>";
        desc += "<span class='card-pos'>[" + p.pos + "]</span>";
        desc += "<a href='#' class='read' data-read='"+p.purekana+"'>[读]</a>";
        var tip = "<span class='card-explain'>" + p.explain + "</span>";
        tip += "<span class='card-pos'>[" + p.pos.slice(0,1) + "]</span>";
        return { tip: tip, desc: desc, read: p.purekana, rid: p.rid }});
    (new easyquiz()).start(quizdata);
  });
});
</script>

<button class="toggle-start btn btn-primary">开始记单词</button>
{% include wordrecite.md %}
