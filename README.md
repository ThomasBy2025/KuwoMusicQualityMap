# KuwoMusicQualityMap

## 项目介绍
该项目主要分析酷我音乐的  
`formats`参数和`N_MINFO`参数，  
获取音乐拥有的全部音质。

## 获取流程
假设需要获取 [妄想感傷代償連盟](https://m.kuwo.cn/newh5app/play_detail/225964339) 的全部音质
- `urlLike = "225964339"`



### 1. 获取`muiscItem`
- 使用 **rid** 获取(支持歌曲链接)
```
async function getMusicInfo(urlLike) {
    var rid = (String(urlLike || "").trim().match(/^(\d+)$/) || [])[1];
    if (!rid) {
        if (!urlLike.match(/kuwo\.cn/i)) {
            return false;
        }
        rid = (urlLike.match(/(\/yinyue\/|\/play_detail\/|[\?\&][rm]?id=(MUSIC_)?)(\d+)/i) || [])[3];
    }
    if (!rid) {
        return false;
    }
    const response = await fetch('http://musicpay.kuwo.cn/music.pay?op=query&action=play&ids=' + rid);
    const data = await response.json();
    return data.songs[0];
}
```
- 通过 **搜索** 获取
```
async function searchMusic(keyword, page=1) {
    const search_url = "http://search.kuwo.cn/r.s?client=kt&all=" +
      encodeURIComponent(String(keyword)) +
      "&pn=" + (page - 1) + 
      "&rn=10&uid=naiy&ver=kwplayer_ar_9.2.2.1&vipver=1" +
      "&show_copyright_off=1&newver=1&ft=music&cluster=0&strategy=2012&encoding=utf8" +
      "&rformat=json&vermerge=1&mobi=1&issubtitle=1";
    const response = await fetch(search_url);
    const data = await response.json();
    return (data.abslist||[])[0];
}
```



### 2. 获取`qualitys`
```
const formatsMap = {...音质列表}

function getQualitys(musicItem, isVideo) {
    let qualitys = [];
    let mvqualitys = [];
    let formats = musicItem.formats.split("|");
    for (let i = 0; i < formats.length; i++) {
        let q = formats[i];
        let f = formatsMap[q];
        if (f.isHide) {
            continue; // 信息不全，不获取该音质
        } else if (f.isVideo) {
            mvqualitys.push(f); // 视频音质
        } else if (f) {

            // 通过 N_MINFO 补全size信息
            let k = f.br.split("k")[0];
            let r = new RegExp('bitrate:' + k + ',format:[^,]+,size:([^;]+)');
            let m = musicItem.N_MINFO.match(r);
            if (m && m[1]) {
                f.size = m[1].replace(/\s*mb/i, " MB");
            }

            qualitys.push(f);
        } else {
            console.log("未收录音质，请反馈 " + q);
        }
    }
    return isVideo ? mvqualitys : qualitys;
}
```



### 3. 获取`url`
```
// 例：quality = getQualitys(musicItem)[0];

const source = "";// 请在其他项目寻找
async function getMediaSource(musicItem, quality) {
    const rid = musicItem.MUSICRID ? musicItem.MUSICRID.split('_')[1].split('&')[0] : (musicItem.rid || musicItem.id);
    const url = 'https://nmobi.kuwo.cn/mobi.s?f=web&type=convert_url_with_sign&source=' + source + '&br=' + quality.br + '&rid=' + rid;
    try {
        const response = await fetch(url);
        const data = await response.json();
        const play_url = data.data.url.split("?")[0];
        return {
            "ekey": data.data.ekey || "",
            "url": play_url.replace("None", "")
        }
    } catch (err) {
        return {
            "ekey": "",
            "url": ""
        }
    }
}
```



## 音质列表
```
// 10000kflac, 7000k[x], 5000k[x], 3000k[dts]?
const formatsMap = {

    // 其他项目常用音质
    "HIRFLAC": {
        "title": "Hi-Res",
        "desc": "录音棚级沉浸临场",
        "br": "4000kflac"
    },
    "ALFLAC": {
        "title": "无损音质",
        "desc": "CD级无损保真",
        "br": "2000kflac"
    },
    "MP3H": {
        "title": "高品音质",
        "desc": "320K-MP3",
        "br": "320kmp3"
    },
    "MP3128": {
        "title": "流畅音质",
        "desc": "128K-MP3",
        "br": "128kmp3"
    },



    // 加密
    "DTSX": {
        "title": "DTS:X",
        "desc": "多维沉浸 声动随心",
        "br": "25000kmmp4",
        "encrypt": true
    },
    "ZPLY": {
        "title": "至臻母带",
        "desc": "还原录音细节 音质提升近一半",
        "br": "20900kmflac",
        "encrypt": true
    },
    "ZPGA714": {
        "title": "臻音全景",
        "desc": "自研空间音频 如同在三维空间",
        "br": "24000kmgg",
        "encrypt": true
    },
    "ZPGA501": {
        "title": "沉浸环绕",
        "desc": "影院级空间感 声临其境的环绕",
        "br": "20501kmflac",
        "encrypt": true
    },
    "ZPGA201": {
        "title": "至臻HiFi",
        "desc": "智能音质增强 补全声场小细节",
        "br": "20201kmflac",
        "encrypt": true
    },



    // 杜比
    "DDJOC768": {
        "title": "杜比 TrueHD",
        "desc": "音质无压缩 完全保留创作者原始音频",
        "br": "11700kmp4"
    },
    "DDJOC640": {
        "title": "杜比 Digital+",
        "desc": "支持7.1声道 提供更清晰一致的环绕声体验",
        "br": "11600kmp4"
    },
    "DDJOC448": {
        "title": "杜比 Digital",
        "desc": "支持5.1声道 高效压缩音频且不损失音质",
        "br": "11500kmp4"
    },
    "AC4256": {
        "title": "杜比 AC-4",
        "desc": "新型音频格式 适配各类音频体验场景",
        "br": "11000kmp4"
    },



    // 无损
    "ZP": {
        "title": "臻音2.0",
        "desc": "智能获取最高音质(不加密)",
        "br": "20000kflac" || "20000kzp"
    },
    "VINYL": {
        "title": "黑胶收录",
        "desc": "48kHz/24bit 支持7.1.4声道",
        "br": "23000kflac"
    },
    "HIRFLAC": {
        "title": "Hi-Res",
        "desc": "录音棚级沉浸临场",
        "br": "4000kflac"
    },
    "ALFLAC": {
        "title": "无损音质",
        "desc": "CD级无损保真",
        "br": "2000kflac"
    },



    // 高品
    "OGGH": {
        "title": "高品音质",
        "desc": "320K-OGG",
        "br": "300kogg"
    },
    "OGG192": {
        "title": "标准音质",
        "desc": "192K-OGG",
        "br": "192kogg"
    },
    "OGG96": {
        "title": "流畅音质",
        "desc": "96K-OGG",
        "br": "100kogg"
    },



    // 标准
    "MP3H": {
        "title": "高品音质",
        "desc": "320K-MP3",
        "br": "320kmp3"
    },
    "MP3192": {
        "title": "标准音质",
        "desc": "192K-MP3",
        "br": "192kmp3" // 波点有，酷我好像没有
    },
    "MP3128": {
        "title": "流畅音质",
        "desc": "128K-MP3",
        "br": "128kmp3"
    },



    // 流畅
    "WMA128": {
        "title": "高品音质",
        "desc": "128K-WMA",
        "br": "128kwma"
    },
    "WMA96": {
        "title": "标准音质",
        "desc": "96K-WMA",
        "br": "96kwma"
    },
    "AAC48": {
        "title": "流畅音质",
        "desc": "48K-AAC",
        "br": "48kaac"
    },



    // 隐藏
    "BCMS": {
        "title": "原唱伴唱",
        "desc": "除母带外 获取加密的最高音质",
        "br": "22000kmgg",
        "encrypt": true, // 该音质需解密
        "isHide": true // 不显示该音质
    },
    "未知": {
        "title": "APE无损",
        "desc": "波点音乐有该音质 酷我未知",
        "br": "1000kape",
        "isHide": true // 不显示该音质
    },
    "AR501": {
        "title": "全景环绕",
        "desc": "模拟5.1环绕声场 让好声音围你而动",
        "br": "10501k", // 不知道后缀
        "isHide": true // 不显示该音质
    },



    // 下面是视频音质，需要vid
    "MV700": {
        "title": "【超清】 720P",
        "desc": "mkv",
        "br": "MV700",
        "isVideo": true
    },
    "MV500": {
        "title": "【高清】 480P",
        "desc": "mkv",
        "br": "MV500",
        "isVideo": true
    },

    // mp4资源
    "MP4BD": {
        "title": "【蓝光】 1080P",
        "desc": "mp4",
        "br": "MP4BD",
        "isVideo": true
    },
    "MP4UL": {
        "title": "【超清】 720P",
        "desc": "mp4",
        "br": "MP4UL",
        "isVideo": true
    },
    "MP4HV": {
        "title": "【高清】 480P",
        "desc": "mp4",
        "br": "MP4HV",
        "isVideo": true
    },
    "MP4": {
        "title": "【标清】 360P",
        "desc": "mp4",
        "br": "MP4",
        "isVideo": true
    },
    "MP4L": {
        "title": "【流畅】 240P",
        "desc": "mp4",
        "br": "MP4L",
        "isVideo": true
    },

    // 虽然加了S，但是并没有加密
    "SMP4BD": {
        "title": "【蓝光】 1080P",
        "desc": "mp4",
        "br": "SMP4BD",
        "isVideo": true
    },
    "SMP4UL": {
        "title": "【超清】 720P",
        "desc": "mp4",
        "br": "SMP4UL",
        "isVideo": true
    },
    "SMP4HV": {
        "title": "【高清】 480P",
        "desc": "mp4",
        "br": "SMP4HV",
        "isVideo": true
    },
    "SMP4": {
        "title": "【标清】 360P",
        "desc": "mp4",
        "br": "SMP4",
        "isVideo": true
    },
    "SMP4L": {
        "title": "【流畅】 240P",
        "desc": "mp4",
        "br": "SMP4L",
        "isVideo": true
    },
}
```



## 其他补充
### 音乐解密
- 请搜索 QMC
### 视频获取
```
// 例：quality = getQualitys(musicItem, true)[0];

async function getVideoSource(musicItem, quality) {
    const vid = musicItem.mvpayinfo?.vid || musicItem.mkvrid;
    if (vid == 0) {
        return "";
    }
    const url = "http://anymatch.kuwo.cn/mobi.s?f=web&type=get_url_by_vid&vid=" + vid + "&quality=" + quality.br;
    try {
        const response = await fetch(url);
        const data = await response.text();
        return data.match(/url=(\S+)/)[1].split("?")[0];
    } catch (err) {
        return "";
    }
}
```