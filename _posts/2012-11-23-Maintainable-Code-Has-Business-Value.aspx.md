---
layout: post
title: Maintainable Code Has Business Value
category: Business
---

I thought this is a no brainer but alas, I should have known better. The web is full of stories about [horrible code](http://thedailywtf.com/) especially about enterprise software, that should be best coded of them all. You can blame it on the developers as they are the ones writing the code, but ask them and they will point the finger (not that finger) at the managers who "are clueless about software developing" and who don't care about anything as long as the software works and it's finished on time.

 The truth is that both parties are to blame, although managers are the main culprits as they can enforce the quality of code. And their problem is that they don't understand that maintainable code has business value.

 But let's define maintainable code: "Maintainable code is code that can be easily understood and modified without generating new bugs". Yes, it's my definition.

 Managers care about 2 things: to finish the project on time and within the budget. Well, if the software is a product which generates revenue or it's an important business asset (used by the business itself to reduce costs or improve productivity) then the project is never finished. Bugs uncovered by users need to be fixed and new features will be added or modified as the client/business needs are changing. And that's where the quality of code will dictate your expenses.

 Simply put, ugly code may work but it's very prone to bugs and any change takes longer than needed because the code is so coupled together that you can't change A without affecting B even if A and B are unrelated features. Fixing bugs usually means breaking some functionality and introducing other bugs. And in time, with every change everything becomes harder and harder, bugs appear more often, the users are unhappy (It doesn't work!!! Stupid software!) and more developers are needed to handle the monster.

 Of course, the new developers need time to learn the code base and to understand it before they can actually do something and in the mean time your users are screaming. In the end, you get to increase costs (from hiring new developers and the good ones probably won't stay too long, unless you're paying them very well and you let them) while the pace of delivering fixes and new features gets slower and slower. And if it's a product you're selling  this means higher development cycle and more time required until you get income.

 But what if the code was maintainable? Then, by default there would be less bugs as there would be automatic testing in place. Changes can be made much faster because of loosely coupled code and with unit tests, any break in functionality or new bugs would be caught instantly. There wouldn't be a need for more developers, in fact it may be the opposite as you wouldn't need a whole team to maintain the application. If you replace a developer, the new one can understand the code base in short time so you get to fix bugs and deliver new features faster.

 Having less bugs and getting fixes and new features will make the users happy. If they are your clients, this means you get to sell them the next version faster. If the software is used internally by the business, then you help your co workers to become more productive and the business more efficient.

 It's important to realise that good code is not just a developers concern because the code is the application. So you need to ask yourself what you want: shoddy software which just works (most of the time anyway) but it is a pain to maintain and develop further or a high quality asset which can be easily adapted to the stakeholders needs. One will increase costs and stress, the other will improve productivity and help increase profits. You decide!


