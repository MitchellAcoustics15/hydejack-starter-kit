---
layout: post
title: Issues with the sonification of novel coronavirus
description: >
  Sonification can be a powerful tool for investigating complex data but sometimes it's just pretty music with an arbitrary past.
tags:       [acoustics, sound science]
# image: /assets/img/blog/example-content-ii.jpg
noindex: true
---

My view of data sonification is that it is an extension or companion to data visualisation. That means it can be an immensely powerful tool which reveals data relationships which are otherwise hidden. <!-- cite a reference on the defined purpose of data visualisation --> However, like standard data visualisation, it can absolutely be misused, misinterpreted, and misapplied. Every data science and statistics handbook I've come across makes it a clear point at the beginning to stress that one of the most crucial steps in analysis is the choosing of appropriate tools. If we plot a histogram where it should instead be a scatter plot, or vice versa, then the actual meaning buried in our data will be obscured by inappropriate (though perhaps not technicall incorrect) analysis. 

Sonification should be treated in the same way. There are several approaches to sonifying a dataset - as there are multiple ways to visualise one - and the selection of the proper method depends on the type of data and what question you are asking of it. 

### Neutral
#### https://www.qub.ac.uk/sarc/research/Covid-19DataSonification65/ 



<!-- The sonification algorithm is a simple mapping between the figures and a sound frequency in Hertz (i.e. the first reported number of 282 infected translates as 282Hz). The result is a complex timbre made up of all the individual figures introduced one second at a time to represent each day since the 21st January (WHO’s Situation Report 1).

To reflect human hearing thresholds (theoretically between 20Hz and 20000Hz), a mechanism was introduced which aims to make audible the rapid speed of contagion. Every time the infected figure goes above 20000, 20000 or multiples of 20000 are subtracted from the ocial figure to bring into hearing range. Everytime this occurs a sharp percussive woodblock sound is heard. The sonification presents two parameters, global number of infected cases (sonified with a simple sine tone) and global number of new deaths sonified with a sawtooth wave, i.e. a more nasal and harsher sounding tone). -->

Personally, I find this style of sonification somewhere in a strange middle ground. I don't see its use for scienctific purposes: I see little feasible reason why new patterns or features would reveal themselves through this window into the data than in a more standard visualisation. It's not particularly complex data - two numbers increasing in a particular fashion. In fact, these particular data seem particularly poorly suited to sonification for the purpose of extracting new information since our ears are not particularly attuned to picking out the sort of frequency changes which result. 

Our auditory system is especially well suited to recognising patterns and ratios. We can hear, for instance, whether a pair of frequencies are a ratio of 3/5 or whether they are 310/500. This is a pretty small difference, and yet even non-musicians are able to tell when that particular ratio is off (although this effect is most prevalent with context; we can notice when one ratio is off if we have heard the correct ratio previously). Likewise, we are very good at recognising rhythmic patterns in sounds, repeating beeps or tones are encoded very quickly into our auditory memory and can be retained there for several weeks subconsciously <!-- cite auditory memory papers from Mercede -->. 

However, this sonification does not demonstrate small differences in 

### Not reviewed
https://www.nytimes.com/video/opinion/100000006819172/the-sound-of-gravity.html


### Good examples of sonification
https://science.thewire.in/the-sciences/gw190412-black-hole-merger-ligo-virgo-harmonics-einstein-prediction/
