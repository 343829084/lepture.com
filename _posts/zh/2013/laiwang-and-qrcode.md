# 不相來往

- date: 2013-11-19 13:34
- tags: thought, wrong

二維碼的濫用正像有聲電影誕生時對聲音的濫用，有時我們需要原始一點的無聲。

----

前同事在微信羣裏發來往的二維碼，我其實也很想加，但是嫌麻煩，就略過了。

我實在不能理解這樣的傳播方式。我至少需要兩部手機，相互掃描二維碼，才能加來往好友，但是我不是土豪。後來聽說不需要兩部手機，把圖片保存下來，來往可以掃描本地圖片，但是我不是阿裏員工。

肉食者鄙，常常做出一些想當然的決策。具體到這件事來說，二維碼是不適合在手機間傳播的。那麼什麼合適呢？我以爲還是 URI 比較好，雖然不夠新潮。比如：

```
laiwang://user/lepture
```

但是微信不會解析該協議，可是微信會解析 HTTP，那我們做一次重定向:

```
http://laiwang.com/app/user/lepture
=> laiwang://user/lepture
```

我知道這樣不夠新潮，可是你確定你追求的是新潮？

這篇上接[《長微博及其它》](http://lepture.com/zh/2013/anti-long-weibo)，喚爲：

> You are doing it wrong !

肉食者鄙，而下不能達上，大公司通病也。