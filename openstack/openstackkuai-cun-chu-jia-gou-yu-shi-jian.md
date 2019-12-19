本文内容来自海云捷迅研发总监吴德新、资深存储研发工程师武宇亭的分享资料《OpenStack块存储架构与实践》，希望对大家有帮助。

  


![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvxkhWCzDHbNpLoerJ15c4vSib9Xqic9QQbRPwtkZ6dwibT7FXzdMickNoJA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvkm1dRI85z4hic43YwXXfOW80PKqfPaMONOK8gscT0NKXiaBciblo7aakw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvM8P023gMic7aVuOG9Xk2YibGzVl0c25cEetR4Nnd1xEsK94m68STMcUA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvIM8v4eoWKg5micjmJGQScUwpa2RAqibjC11yBBKeGEhgiamjvTJJC7JNA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvvWicWhvkkvicZA3DtxG7Jh50libcS937w8LsZ3J9jVeibyOc5VTykBAJzA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvic850xSmfEjsrtA8nLDLvAHyOBQ9VKHJu8WOyh7KNWyYkf9kMMRViaDQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvrffMx6yXe041pO3iczp0SsNnTBlq2QVsTc7eBL2uqh7TI4Q1YU8ge6A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvL3iaPVEIr45iaEG6aicAN3AIB7nUtrzVLWhI39nYRzRFU6icma7ZniarfEg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvZlYT06LposaPdwshxotXLyiaSbEv1MpzQqfWicRLcjphIYico8ibUDLEAg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvibZzlbN24tZNrQPRzIE1s4OSg1g1ElYk7TnWFYTS4h7bQFIhf2sJeEA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  


![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvouRyiaSAC8pc7L23AjZISg8YXUicTlf780qCFjHVpOnRJ8cPWFbw3e4g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvdvl8mpnL239OxfQf0Slib8icl1vLx7BY3wtuT0WicuS2foZ62DZWJskibQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvic9FjekXp4k4SH30Ng2NZ5oAxAPGic4CE0PAYyR6Dq6JJ48nzbnsniavQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvJxWGweYc4DeS3HUbgvvEEN8GG7JYPjBAnJUQSZy490ibLtv2XeFJ1Ew/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvl8ibnfiasibSlRTOnVIWToObcISGaC54rZLsqTHlueRia65DMgBePsvicUA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvhNULtcKmNyG1D2JpSw66K5qukuYZbemibI7afoHrVk3ibs6ibdgB1YECQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvzgaBJG3d47GooE8YhPyCj2ZLXR5Ticxx1kvzhiaUT0FksJbBicHmvo3Nw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvWrOIK2cDzibF2qHw4p0VyNWO0kIu4DsGurM7juowMiaichmaxZbvGpFoA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvpXKDqfKVjJyEabECbTOtdibeyicoebxpoIv8sjaSP0nvC9nCxee3y0Ng/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvOLpBFldYDVVVmmKvy6cFf3xjrw4RcL7ZNLXbpRgAiaO5rVNyKCc5OvA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvqVCAicUAlSaLHu66HYxYSvqNGhdeZSywYAWkyBGUhBIB7W1bHbuib6Hg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvQIDEiaY09L2B3wO5SLAbBxXpvl6Oxdx4dBhTQ5awQeohDKCsIfkIUJQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvhGjRPWuS6GwzwMiccLHTy9TU0YLR0X3RfKWdtibPRM4V5lG04EgkvP6A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvFk1e4ywbp3MrBmPEr2ejm9CxWKTTFj17xczyT87rdj6oJWzPmLwX5w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/u3ZiaMm7TCFWR7Z9nysc2BULtx5iaPugPvWZkSyaVFu6KXrbkb3ViauYepRQzHZoR6KG4GibtayVyPmeUyVRDVGaxA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

