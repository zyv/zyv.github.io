---
layout: post
title: XQuery saves hacker's life while shopping for bedding
categories:
- tech
- xquery
---

A couple of days ago I was confronted with an unpleasant discovery in bed, which once again has made me acutely aware of the frailty of life. Specifically, I was looking at a glaring hole in my soon to be a decade old bed sheets.

"How annoying!" I thought, being presented with a very tangible physical problem, which cannot be immediately solved with an elegant one-liner shell script... Oh, wait! In this day and age IKEA must certainly have an online shop, so I could just pick a new set of linen and get it shipped to my doorstep.

Moreover, in spite of an overwhelming multitude of products available for purchase, one can apply a simple set of criteria, which would invariably lead to an optimal solution to this otherwise _NP_-hard problem:

  * Assuming that the price is still any indicator of quality in our post-modern world, products should be sorted by price, descending
  
  * Products should be filtered by type and grouped by family, allowing for colour and design variations to be easily assessed
  
  * Designs featuring variegated colouring of any kind or anathematic striping patterns should be immediately discarded from consideration

The rest should _purely_ amount to `map buy $ take 2 products`.

Sure enough, however, this quickly turned out to be one of those brilliant plans that are notoriously easier to verbalise than execute.

An enterprising adventurer who chances to look for sheets on the [IKEA website](https://www.ikea.com/de/de/cat/bettwaesche-tl004/) is immediately confronted with the lack of basic hygiene facilities like sorting by price, not even speaking of filtering by type. The products are displayed in a seemingly random order, which precludes any organised attempts to make sense of what design variations are available for a particular type.

Fortunately, no amount of corporate cargo-agile website-building idiocy can dissuade a determined hacker on the verge of perdition from buying new bedding. They must have an API, which can be queried to obtain a list of products, so that one can hack up a [JMESPath](http://jmespath.org) expression and be done with it, right? **Wrong!** They do have one indeed, but it is one of those special [dangerous street APIs](https://www.youtube.com/watch?v=wTqsV3q7rRU) returning pre-rendered HTML snippets, which you are supposed to shove up your DOM.

Here is the kind of sodomy that one would typically have to deal with:

```html
<div class="product-compact"
  data-ref-id="10357259"
  data-price="11.99"
  data-currency="EUR"
>
  <div class="product-compact__spacer">
    <a href="https://www.ikea.com/de/de/p/dvala-bettlaken-weiss-10357259/">
      <div class="product-compact__image-container">
          <div class="product-compact__image">
            <div class="range-image-claim-height" style="padding-bottom: 99.95%;">
              <img src="..." alt="IKEA DVALA Bettlaken">
            </div>    </div>
        </div><span class="product-compact__name">DVALA</span>
      <span class="product-compact__type">
        Bettlaken,
          <span class="product-compact__description">240x260 cm</span></span><span class="product-compact__price">
    <span class="product-compact__price__value">11.99</span>  </span>    </a><a href="https://www.ikea.com/de/de/p/dvala-bettlaken-weiss-10357259/">
        <span class="product-compact__gpr-disclaimer disclaimer">
          Weitere Farben/ Ausführungen vorhanden
        </span></a>  </div>
</div>
```

Challenge accepted!™ Let us fire up BaseX and [put the impertinent Swedes back in check](https://en.wikipedia.org/wiki/Battle_of_the_Neva) by obtaining some actionable insights from their ~~Big~~^WGarbage Data:

```console
zaytsev:~ zaytsev$ brew cask install adoptopenjdk
zaytsev:~ zaytsev$ brew install basex
zaytsev:~ zaytsev$ basexgui
```

```xml
declare variable $doc := html:parse(fetch:binary('https://www.ikea.com/de/de/cat/bettwaesche-10651/'));
 
(: declare variable $typeFilter := ''; :)
declare variable $typeFilter := 'Bettwäscheset';
        
declare function local:trim($arg) {
   replace(replace($arg,'\s+$',''),'^\s+','')
};
 
declare function local:parseItem($item) {
  let $cleanName := $item//span[@class='product-compact__name']/text()
  let $cleanTypes := $item//span[@class='product-compact__type']//text() ! local:trim(.)
  let $cleanPrice := $item/@data-price/data()
  let $cleanLink := $item//a[1]/@href/data()
  let $cleanRemarks := $item//span[contains(@class, 'disclaimer')]/text() ! local:trim(.)
  return
    <Item>
      <Name>{ $cleanName }</Name>
      <Type>{ string-join($cleanTypes, ' ') }</Type>
      <Price>{ $cleanPrice }</Price>
      <Link>{ $cleanLink }</Link>
      <Remarks>{ $cleanRemarks }</Remarks>
    </Item>
};

let $items := $doc//div[@class='product-compact']
let $results := 
  for $item in $items
  let $data := local:parseItem($item)
  where contains($data/Type, $typeFilter)
  order by number($data/Price) descending
  group by $name := $data/Name
  return
    <Product name="{ $name }" variants="{ count($data) }">{
      for $variant in $data
      order by number($variant/Price) descending
      return <Variant>{ $variant/node()[not(name(.)='Name')] }</Variant>
    }</Product>
return
  <Results
    filtered-products="{ count($results) }"
    filtered-items="{ count($results//Variant) }"
    total-items="{ count($items) }"
    >
    <Types>{
      for $type in distinct-values($results//Type)
      order by $type
      return <Type>{ $type }</Type>
    }</Types>
    <Products>{ $results }</Products>
  </Results>
```

Now, this does look like something meaningful:

```xml
<Results filtered-products="12" filtered-items="22" total-items="24">
  <Types>
    <Type>Bettwäscheset, 2-teilig, 140x200/80x80 cm</Type>
    <Type>Bettwäscheset, 2-teilig, 155x220/80x80 cm</Type>
    <Type>Bettwäscheset, 3-teilig, 240x220/80x80 cm</Type>
  </Types>
  <Products>
    <Product name="PUDERVIVA" variants="1">
      <Variant>
        <Type>Bettwäscheset, 2-teilig, 155x220/80x80 cm</Type>
        <Price>59.99</Price>
        <Link>https://www.ikea.com/de/de/p/puderviva-bettwaescheset-2-teilig-hellgelb-80431595/</Link>
        <Remarks>Weitere Farben/ Ausführungen vorhanden</Remarks>
      </Variant>
    </Product>
    <Product name="SÄCKBUSKE" variants="4">
      <Variant>
        <Type>Bettwäscheset, 2-teilig, 155x220/80x80 cm</Type>
        <Price>49.99</Price>
        <Link>https://www.ikea.com/de/de/p/saeckbuske-bettwaescheset-2-teilig-grau-00448397/</Link>
        <Remarks>Weitere Farben/ Ausführungen vorhanden</Remarks>
      </Variant>
      <!-- ... -->
    </Products>
</Results>
```

After dumping the whole DOM to file and some more fiddling with XQuery, the fog of war finally starts clearing up!

By now it is pretty obvious that I need a PUDERVIVA or SÖMNTUTA (actually, one of each to be on the safe side), as well as one SÄCKBUSKE to match PUDERVIVA with a backup BRUNKRISSLA to match SÖMNTUTA.

Relieved by the situation returning under control, let us apply the same procedure to other kinds of household textiles. How about buying a bunch of BINNAN towels as well while we are at it?

***

I am very impressed by the fact that although the same results could have been achieved by hacking together a throw-away Python script leveraging `lxml`, in this case, it is pretty clear indeed, that nothing matches the fun, power and elegance of XQuery.

Thanks to Hans-Juergen Rennau for giving me a glimpse into this   wondrous world years aback!
