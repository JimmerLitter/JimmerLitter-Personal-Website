---
title: "Compounding Effects of Tolerances in a Cycloidal Gearbox"
date: 2026-08-26
draft: false
slug: "cycloidal-gearbox-3d-printing-tolerances"
aliases: ["/posts/blog-003/"]
showToc: false
TocOpen: false
tags: ["gears", "3d printing", "mechanical", "engineering"]
categories: ["blog"]
description: "Building my first cycloidal gearbox on a 3D printer, and learning the hard way how much tolerances matter: sawing roller pins by hand, hammering in bearings, and a mysterious 2 mm gap."
---

"Tolerances are insane, especially if you're 3D printing" is what someone told me when I said I was working on a cycloidal gearbox. I couldn't agree more. This was the first project where I was punished every time I didn't get my tolerances precise enough. Fusing 3D-printed pieces with metal was incredibly frustrating, and on a project where there were so many complex pieces, each mistake compounds.

I chose the cycloidal drive as my first gearbox because I was very interested in the motion. I still remember I came across a YouTube video that was alluding to getting higher torque gears without teeth. Needless to say, it was an unskippable title, and after watching it, I was determined to get my hands dirty with a 3D-printed design.

With limited hours of access in my local makerspace, this little gearbox actually took me nearly 2 weeks to even be done, and it's not even good yet. Here's why it took me so long:

I didn't start off CADing the whole thing. I copied tutorials online and their respective design files since I was a noob. I prioritized getting this done and just building, so I opted to bring what the tutorial gave me. This was a big mistake because there were some measurements I needed that were difficult to measure with calipers or rulers or anything. I also didn't have access to the actual 3D model, only the pure STL. Not owning the measurements cost me a lot of time doing trial and error. Sometimes the holes were too small and I had to add negatives in the slicer for every single hole; super duper tedious and annoying. Definitely, when I run a second iteration, I will model it from the ground up, but I also feel like I had to learn this lesson the hard way so I don't get lazy again. If it's any consolation, I was just trying to get this done before I had to leave the country.

A cycloidal drive is composed of the housing, roller pins, input and output shafts, eccentric shaft, and the cycloid disks. Take the roller pins, for example: each one has 2 bearings and some distance rings between each bearing, which are all held together by a metal pin. The precision of the pin caused me many problems. I didn't have access to any metal-cutting machines, so I had to saw each piece out of a larger rod. If the pins are even 0.5-1 mm off, they don't fit cleanly into the housing and lid. So while I was sawing the rod, I didn't realize that me accounting for 2 mm of headway each cut was still not good enough. Although a good workout, sawing also leaves uneven edges that don't fit easily into a bearing. I needed a serious hand massage after finally finishing filing all the pins flat, but the fit was still difficult and I still needed a hammer to persuade the bearings in. The hammer massage was well worth it though.

Tolerance issues seeped their way into the lid assembly as well. The eccentric shaft was easier, but the two bolts holding everything together were just a bit too long, which made it difficult to bolt everything else together since the gearbox couldn't be made flat.

The final assembly also took much persuasion, probably due to all the accumulating tolerance failures, and I still have no clue why there's a 2 mm gap between the lid and the housing. Distance rings kept falling off because they were fractions of a millimeter too big. The cycloidal disks were just big enough that they slipped slightly on their bearings, which caused them to get too close to the other disk.

What I constantly grappled with was how tight I wanted the fit to be. Too tight and I spend so long getting it to fit; too loose and I lose time anyway trying to get things to stay where they are. I suppose all that extra time should be spent printing an assortment of pieces in increasing size and pinpointing the exact size of the best piece possible in increments of fractions of a millimeter, maybe less. Therefore, with all I learned, it'll definitely be a lot easier in a second iteration. But I was definitely discouraged from doing so since I didn't want someone to snatch away the community 3D printer if I was away for 5 minutes doing some quick sizing. The hours were also pretty shit. Pictures will be on my GitHub.

I can't even imagine doing this at scale with thousands of parts and integrating with [motors]({{< relref "blog-002" >}}). There really are levels to this. But it's only a matter of time until I get there.

TLDR:

1. Do a full CAD model of your project; don't be lazy.
2. Design around the parts that are the hardest to fit.
3. Run lots of trial pieces within a range of sizes with small changes to make sure components assemble properly.
4. Keep an engineering notebook of the changes in sizing for a piece; note what you settled on to make the second time way faster.
