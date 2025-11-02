import { createTestEnv, req, kitty } from "utils"

interface IVod {
  url: string
  vod_name: string
  vod_pic: string
  vod_remarks: string
  vod_blurb: string
}

export default class Ysxqcc implements Handle {
  getConfig() {
    return <Iconfig>{
      id: "ysxq.cc",
      name: "播剧网",
      type: 1,
      nsfw: false,
      api: "https://www.ysxq.cc",
    }
  }
  async getCategory() {
    return [
      { "text": "首页", "id": "/" },
      { "text": "电影", "id": "1" },
      { "text": "连续剧", "id": "2" },
      { "text": "综艺", "id": "3" },
      { "text": "动漫", "id": "4" },
      { "text": "短剧", "id": "60" }
    ]
  }
  async getHome() {
    const cate = env.get<string>("category")
    const page = env.get("page")
    if (cate == "/") {
      const $ = kitty.load(await req(env.baseUrl))
      const banner = $(".slid-e-list .slid-e-list-box").toArray().map<IMovie>(item => {
        const cover = $(item).find(".swiper-lazy").attr("data-background") ?? ""
        const title = $(item).find(".slide-info-title").text().trim()
        const desc = $(item).find(".slide-info").text().trim()
        const remark = $(item).find(".slid-e-type").text().trim()
        const id = $(item).find(".tim-bj.collection").attr("data-id") ?? ""
        return { title, cover, desc, remark, id }
      })
      const table = $(".theme2 .box-width.wow").toArray().map<IHomeContentItem | null>(item => {
        const title = $(item).find(".title .title-h").text().trim()
        const isCard = $(item).find(".swiper-wrapper").length > 0
        const videos = $(item).find(".public-list-box").toArray().map<IMovie>(subItem => {
          const a = $(subItem).find("a")
          const title = a.attr("title") ?? ""
          const id = a.attr("href") ?? ""
          let cover = env.baseUrl + a.find("img.lazy").attr("data-src")
          const remark = a.find(".public-list-prb").text().trim()
          return { id, title, cover, remark }
        })
        if (!videos.length) return null
        return {
          type: isCard ? "card" : "list",
          title,
          videos
        }
      }).filter(item => !!item)
      return <IHomeData>{
        type: 'complex',
        data: [
          {
            type: 'banner',
            videos: banner,
          },
          ...table,
        ]
      }
    }
    const resp: { list: Array<IVod> } = JSON.parse(await req(`${env.baseUrl}/index.php/ds_api/vod`, {
      method: "POST",
      noCache: true,
      data: {
        type: +cate,
        level: 0,
        by: 'time',
        page,
      },
    }))
    return resp.list.map<IMovie>(item => {
      return {
        id: item.url,
        title: item.vod_name,
        cover: env.baseUrl + item.vod_pic,
        remark: item.vod_remarks.trim(),
        desc: item.vod_blurb.trim(),
      }
    })
  }
  async getDetail() {
    const id = env.get<string>("movieId")
    const $ = kitty.load(await req(`${env.baseUrl}${id}`))
    const desc = $("#height_limit").text().trim()
    const tabs = $(".anthology-tab .swiper-slide").toArray().map(item => {
      const text = $(item).text().replace(/\d*$/, "")
      return text.trim()
    })
    const videos = $(".anthology-list .anthology-list-box").toArray().map(item => {
      return $(item).find("li").toArray().map(subItem=> {
        const id = $(subItem).find("a").attr("href") ?? ""
        const text = $(subItem).text()
        return { id, text }
      })
    })
    const playlist = tabs.map((tab, index) => {
      const value = videos[index]
      return <IPlaylist>{
        title: tab,
        videos: value,
      }
    })
    return <IMovie>{ desc, playlist }
  }
  async getSearch() {
    const wd = env.get<string>("keyword")
    const page = env.get<string>("page")
    const url = `${env.baseUrl}/vodsearch/${wd}----------${page}---.html`
    const $ = kitty.load(await req(url))
    return $(".box-width .row .vod-detail").toArray().map(item=> {
      const img = $(item).find("img.lazy")
      const cover = env.baseUrl + img.attr('data-src')
      const title = img.attr('alt') ?? ""
      const remark = $($(item).find(".slide-info-remarks").toArray()[0]).text().trim()
      const id = $(item).find('a').attr('href') ?? ""
      return { id, cover, title, remark }
    })
  }
}

const env = createTestEnv("https://www.ysxq.cc")
const call = new Ysxqcc();
(async () => {
  const cates = await call.getCategory()
  env.set("category", cates[0].id)
  env.set("page", 2)
  const home = await call.getHome()
  env.set("keyword", "我能")
  env.set("page", 1)
  const search = await call.getSearch()
  if (!Array.isArray(home)) return
  env.set("movieId", "/voddetail/150314.html" /*search[0].id*/)
  const detail = await call.getDetail()
  debugger
})()
