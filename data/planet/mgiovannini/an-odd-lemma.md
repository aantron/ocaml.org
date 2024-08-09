---
title: An Odd Lemma
description: "While proving that every monad is an applicative functor, I extracted
  the following derivation as a lemma:      fmap f \u2218 (\u03BBh. fmap h x) \u2261
  { ..."
url: https://alaska-kamtchatka.blogspot.com/2012/07/odd-lemma.html
date: 2012-07-17T17:36:00-00:00
preview_image:
authors:
- "Mat\xEDas Giovannini"
source:
---

While proving that every monad is an applicative functor, I extracted the following derivation as a lemma:


  fmap f ∘ (λh. fmap h x)
≡ { defn. ∘, β-reduction }
  λg. fmap f (fmap g x)
≡ { defn. ∘ }
  λg. (fmap f ∘ fmap g) x
≡ { Functor }
  λg. fmap (f ∘ g) x
≡ { abstract f ∘ g }
  λg. (λh.
