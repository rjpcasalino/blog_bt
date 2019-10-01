---
title: 'Daily Update'
layout: post
---

- [Hong Kong Protester Is Shot By Police As Clashes Over Democracy Take A Dark Turn](https://www.npr.org/2019/10/01/765971927/hong-kong-protester-reportedly-shot-by-police-during-mass-demonstrations)

Meanwhile we are snarled in impeachment intrigue.

- [In a wide-ranging interview with Morning Edition's Steve Inskeep, Cui Tiankai, China's ambassador to the U.S., discusses 70 years of Communist Party rule, President Xi Jinping's move to do away with presidential term limits and the Hong Kong protests.](https://www.npr.org/2019/10/01/765833918/transcript-nprs-interview-with-china-s-ambassador-to-the-u-s)

> "You see all the teachings, the values by Confucius, by other very wise men in the past; they are in the DNA of the Chinese nation. You have to understand this. But of course you should also have a close look at what happened in the last two centuries or so since the Opium War in 1840. You see a very proud ancient civilization was invaded, exploited, oppressed by foreign powers. This has left a very deep impact on the mindset of the Chinese nation. So for over 100 years, generation after generation of Chinese try their best to modernize the country, to revitalize, rejuvenate an ancient civilization. Try to have an ancient civilization modernize while still keeping some of its essential elements, keeping its own tradition. I think of the, the founding of the Communist Party was just against this background and throughout — you see, in about two years' time we will be commemorating the one hundredth anniversary of the founding of the Communist Party."

> "This party was born in China. It grew out of Chinese soil. Of course we learn a lot of things. We sort of imported Marxism, but we always say we are doing something with Chinese characteristics. It means the Chinese culture, Chinese tradition is always there. So the Communist Party in China is very different from the Communist Party of the former Soviet Union or maybe in some other countries. It is very Chinese. Of course it is a Communist Party. But I think in the history of mankind people are always striving for better life, equality, freedom or these good things, and people may follow a different path, may have different ways of achieving the goals, and maybe some people believe in this ideology or philosophy. Others believe in others. I think that people could try. People of all countries in the world, they could try, they could explore their own ways of running their country, achieving modernization, economic development, freedom for the people and build their country strong, prosperous."

"freedom and these good things"

Cold War II, folks. 

Ordered a System76 branded nuc yesterday. Fairly excited to put NixOS on that bad boy.

- [Joining Apple Computer:Bill Atkinson](https://www.folklore.org/StoryView.py?project=Macintosh&story=Joining_Apple_Computer&sortOrder=Sort+by+Date)

>"Inspired by a mind-expanding LSD journey in 1985, I designed the HyperCard authoring system that enabled non-programmers to make their own interactive media. HyperCard used a metaphor of stacks of cards containing graphics, text, buttons, and links that could take you to another card. The HyperTalk scripting language implemented by Dan Winkler was a gentle introduction to event-based programming. Steve Jobs wanted me to leave Apple and join him at Next, but I chose to stay with Apple to finish HyperCard. Apple published HyperCard in 1987, six years before Mosaic, the first web browser."

testing in Go (using Gin) is sorta fun:

```
package main

import (
 "net/http"
 "net/http/httptest"
 "os"
 "testing"

 "github.com/stretchr/testify/assert"
)

func TestStatusRoute(t *testing.T) {
 router := setupRouter()

 w := httptest.NewRecorder()
 req, _ := http.NewRequest("GET", "/api/v3/status", nil)
 req.Header.Set("x-api-key", os.Getenv("API_KEY"))
 router.ServeHTTP(w, req)

 assert.Equal(t, 200, w.Code)
 assert.JSONEq(t, `{"message": "OK"}`, w.Body.String())
}

```
