---
theme: 'default'
transition: fade-out
lineNumbers: true
highlighter: shiki
---
# Maybe in **Typescript**

Sam Broster 2023


<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
layout: intro
---

<style>
p {
  font-size: 1.5em;
}
</style>

# Null

> I call it my billion-dollar mistake ... has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage ...

<div align=right>— C. A. R. Hoare</div>

<!--
* Read quote
* A function that can return null needs to checked
  * checking is optional! 
  * null can propogate 
  * error when value used NOT when function called
-->

---

# Introducing Maybe

<div class="grid grid-cols-[60%,40%] gap-4">
  <div>

A `Maybe` can be one of two types:
* `Just` contains a value that is of type `T`
* `Nothing` is empty 

  <br> 

```ts
export const Nothing = (): Nothing => {
  type: MaybeType.Nothing,
})
  
export const Just = <T> (value: T): Just<T> => ({
  type: MaybeType.Just,
  value,
})
```

  </div>
  <div>

```ts {1-9|1-4,10-13|5-}
enum MaybeType {
  Just = 'maybe-type__just',
  Nothing = 'maybe-type__nothing',
}

interface Just<T> {
  type: typeof MaybeType.Just
  value: T
}

interface Nothing {
  type: typeof MaybeType.Nothing
}

export type Maybe<T> = Just<T> | Nothing
```

</div>
</div>

<!--
* Motivation - non-optional checking
* Maybe is more explicit than relying on TS to check nulls
* Implementation ...
-->

---
layout: center
---

# A Maybe Example

```ts {-7|9-}
const getPetById = (id: string): Maybe<Pet> => {
  const pet = query({petId: id})
  if (!pet) {
    return Nothing()
  }
  return Just(Pet)
}

const updatePetById = (id: string, age: number): Maybe<Pet> => {
  const maybePet = getPetById(id)
  if (maybePet.type === MaybeType.Nothing) {
    return Nothing()
  }
  const updatedPet = putPet({ ...maybePet.value, age })
  return Just(updatedPet)
}
```

<!--
* getPetById - what can it return
* updatePetById - explicitly has to check before it can use the value
* a bit verbose
 -->

---
layout: center
---

# Utilities

* *maybeWithDefault* - <small> use a default value if **Maybe** is **Nothing** </small>
  ```ts
  maybeWithDefault = <T>(m: Maybe<T>, defaultValue: T): T =>  {...}
  ```

* *getOrThrow* - <small> throw an exception if **Maybe** is **Nothing** </small>
  ```ts
  getOrThrow = <Y>(m: Maybe<T>, throwFunc: () => void)) : T => {...}
  ```

* *maybeAndThen* - <small> perform and action if a value is a **Just** value </small>  

* *maybeMap* - <small> perform an action on all **Just** values in an array </small>

---
layout: center
---

# Comparison of maybeAndThen

## **Before**
```typescript
const updatePetById = (id: string, age: number): Maybe<Pet> => {
  const maybePet = getPetById(id)
  if (maybePet.type === MaybeType.Nothing) {
    return Nothing()
  }
  const updatedPet = putPet({ ...maybePet.value, age })
  return Just(updatedPet)
}
```
<br>

## **After**
```typescript
const updatePetById = (id: string, age: number): Maybe<Pet> => {
  return maybeAndThen(
    getPetById(id)
    (pet) => {
      const updatedPet = putPet({ ...maybePet.value, age })
      return Just(updatedPet)
    }
  )
}
```

<!--
* Coontrast previous example 
* syntactic sugar - Less chance to get comparison wrong
* Don't have to use anonymous function - even simpler 
 -->

---
layout: center
---

# maybeAndThen

```typescript
const maybeAndThen = <T, R>(m: Maybe<T>, func: (t: T) => Maybe<R>): Maybe<R> => {
  if (m.type === MaybeType.Nothing) {
    return Nothing()
  }
  return  func(m)
}
```
<br>

* Input: `Maybe<T>`
* Output: `Maybe<R>`
* Function that converts `T` to `Maybe<R\>` 

<!--
* Explain type signature & function
 -->

---
layout: center
---

# Arrays of Maybes

```ts {-9|11-|all}
const updatePetsByIds = (ids: string[], age: number): Maybe<Pet>[] => {
  return maybeMap(
    ids.map(getPetById),
    (pet) => {
      const updatedPet = putPet({ ...maybePet.value, age })
      return Just(updatedPet)
    }
  )
}

const maybeMap = <T, R>(ms: Maybe<T>[], func: (t: T) => Maybe<R>): Maybe<R>[] => {
  return ms.map((m) => maybeAndThen(m, func))
}
```

<!--
* Extension of update Pet function so support multiple updates to demo maybeMap
* maybeMap easy to implement
 -->


---
layout: center
---

# Final Thoughts

* `Maybe` makes returning and checking for `null` explicit
* Utility functions add syntactic sugar
* Reduced value by lack of *polymorphic* functions in Typescript

---
layout: end
---