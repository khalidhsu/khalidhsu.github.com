Title: 定時重啓非智能路由
Date: 2014-02-07 22:06  
Tags: router,gae  
Category: 2014-02  
Slug:  auto-reboot-router   
Author: Khalid Hsu  
  
最近路由智能的概念，讓這個被遺忘在客廳角落積滿灰塵的破盒子重新煥發光彩，搖身變成各互聯網公司眼中流量入口的必爭之地。  
不過本文所述的路由器，並非近期大紅大紫的智能路由。  
而恰恰是因爲我手頭這款路由器極其不智能，才有此無聊折騰。  
首先，感謝手頭這款路由器。它陪伴我度過了2014的春節。期間它超高負荷，運作不斷，服務了大大小小的各種設備，包括我的PC，各位親戚朋友的手機等等。如此艱難困苦它也沒有半句怨言，實屬難得。  
不過話說回來，這台路由器還真挺搓的。至於它有多搓、哪裏搓，我沒有這個精氣神去細數。它是哪個牌子哪個型號的，我也不黑它。它滿足不了我，不代表滿足不了其他人。  
但是，它有一個明顯的缺點讓我特不爽特想吐槽，就是他的持久力不夠！有點類似內存泄漏的症狀，運作時間一長就會神經兮兮，連後臺管理頁面都無法訪問。此刻只能無奈地給它來個硬重啓。然後它立即又沒心沒肺的滿血復活，甚至比吃了菠菜的大力水手還要猛N倍，完全沒有剛剛的痛苦。   
這樣看來，要解決這個問題也不是沒辦法的，每天給它重啓一遍就行。此神經病雖未能根治，但湊合着也還能過日子。  
當然，每天都屁顛屁顛的來到路由面前按一下重啓，然後又屁顛屁顛的回到電腦面前，這事情可不是人幹的，至少我不會這樣幹。  
實際上我的第一個解決辦法是，在X寶上買了一個定時開關，設定給路由器的開關時間就OK了。不過這可憐的定時開關，沒活幾天就因爲一個意外而短路一命嗚呼了，劇情比韓劇還要狗血。  
更傷心的是，臨近過年X寶上的賣家都說要等春節過後才發貨。擦，無奈之下就有了第二個方法。  
  
方法二其實也很簡單，路由器的後臺管理頁面一般都提供重啓功能。咱就直接寫個腳本模仿一下後臺操作。用curl那就是：  
  
```bash  
  
curl 'http://192.168.1.1/userRpm/SysRebootRpm.htm?Reboot=%D6%D8%C6%F4%C2%B7%D3%C9%C6%F7' -H 'Authorization: Basic dXNlcjpwYXNzd29yZA=='  
# 常見的路由器都用HTTP基本認證，也就是Authorization這個header，base64編碼  
  
```  
  
只要執行一下這行命令就能重啓路由了。不過如果赤裸裸的照做，那用戶體驗也太差了。重啓一下路由我得等個把分鐘才能連上wifi。時間比金錢重要，沒理由把青春揮霍在這。  
  
我覺得在某一個時間讓路由器自動重啓更好。對我來說，凌晨5點半是個不錯的時間，因爲我通常不會熬夜到這個時間。不過此時電腦關了，crontab無效。我也不喜歡讓手機，平板承擔如此苦逼的定時任務，只好辛苦一下神聖的gae了。在此感谢它给我，還有和我一樣飽受專制壓迫的廣大網民带来的精神自由。  
  
說明一下，要通過gae來重啓路由，你得先在路由中設置允許遠程web管理。另外，因爲每次重啓都會變動IP，所以你還得開啓路由的動態DNS（DDNS）。  
  
  
下面，建立一個gae工程（我直接在sdk的demo上稍作改動）：  
autorouter.py：  
  
```python  
  
import cgi  
import webapp2  
from google.appengine.api import urlfetch  
  
def reboot_router():  
  url='http://yourname.eicp.net:8123/userRpm/SysRebootRpm.htm?Reboot=%D6%D8%C6%F4%C2%B7%D3%C9%C6%F7'  
  header_authorized='Basic dXNlcjpwYXNzd29yZA=='  
  result = urlfetch.fetch(url=url,  
                          method=urlfetch.GET,  
                          headers={'Authorization': header_authorized })  
  
class MainPage(webapp2.RequestHandler):  
  def get(self):  
    reboot_router()  
  
app = webapp2.WSGIApplication([  
  ('/', MainPage),  
], debug=True)  
  
  
```  
也就是直接用urlfetch發出一個一樣的http請求，目標域名是我在花生殼上註冊得來的。  
  
下面设置定时任务，每天凌晨五點半之前我應該都能按計劃離開電腦去睡覺。別忘了還有設置timezone。  
cron.yaml：  
  
```yaml  
  
cron:  
- description: auto reboot the router  
  url: /  
  schedule: every day 05:30  
  timezone: Asia/Hong_Kong  
  
```  
  
  
最好更改一下訪問權限，如果不想其他人幫忙重啓路由。  
handlers增加login: admin  
  
app.yaml：  
  
  
```yaml  
  
application: khalidrouter  
version: 1  
runtime: python27  
api_version: 1  
threadsafe: yes  
  
handlers:  
- url: .*  
  script: autorouter.app  
  login: admin  
libraries:  
- name: webapp2  
  version: "2.5.2"  
  
```  

[注：上述文件已經放到github中 。](https://github.com/khalidhsu/autorouter) 



最後還得注意一點，那就是路由器的撥號連接最好設置成自動連接，不要設置成“按需連接"。以保證gae發出重啓指令時候，路由是在線的。  
好吧，爲了這破路由，傾注了一池廢話。要是它能刷openwrt，一切都不必如此。   
  
  
  
