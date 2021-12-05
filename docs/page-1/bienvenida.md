# Bienvenida

Construye aplicaciones reactivas con las herramientas de Rails que ya conoces y amas

### &#x20;<a href="#what-is-stimulusreflex" id="what-is-stimulusreflex"></a>

**Una nueva forma de crear interfaces web modernas y reactivas con Ruby on Rails.**

Ampliamos las capacidades de ambos [Rails](https://rubyonrails.org) y [Stimulus](https://stimulusjs.org), interceptando las interacciones del usuario y pasándolas a Rails a través de websockets en tiempo real. Estas interacciones son procesadas por acciones Reflex que cambian el estado de la aplicación. La página actual se vuelve a renderizar rápidamente y los cambios se envían al cliente utilizando [CableReady](https://cableready.stimulusreflex.com). La página es entonces es Mutado, [morphed](https://github.com/patrick-steele-idem/morphdom). para reflejar el nuevo estado de la aplicación. Todo este viaje de ida y vuelta nos permite actualizar la interfaz de usuario en 20-30ms sin parpadeos ni costosas cargas de página.

Esta arquitectura elimina la complejidad impuesta por los frameworks full-stack frontend sin abandonar experiencias de usuario reactivas de alto rendimiento, [high-performance reactive user experiences](https://www.youtube.com/watch?v=SWEts0rlezA\&t=214s). Con StimulusReflex, los equipos pequeños pueden hacer grandes cosas más rápido que nunca. Te invitamos a explorar una nueva alternativa a la "Single Page App / Aplicación de una sola página"  (SPA).

**Involúcrate.** ¡Juntos somos más fuertes! Únete a nosotros en nuestro [Discord.](https://discord.gg/stimulus-reflex)

[![](https://img.shields.io/discord/629472241427415060)](https://discord.gg/stimulus-reflex)

[​](https://discord.gg/stimulus-reflex)​

### &#x20;<a href="#why-should-i-use-stimulusreflex" id="why-should-i-use-stimulusreflex"></a>

¿No sería estupendo que pudiera **centrarse en su producto** en lugar del ruido técnico introducido por el JavaScript moderno? Con StimulusReflex, podrá **entregar proyectos rápidamente, con equipos más pequeños** y redescubrir el placer de la programación.

### &#x20;<a href="#goals" id="goals"></a>

* Permitir que los equipos pequeños hagan grandes cosas, más rápido 🏃🏽‍♀️
* Aumentar la felicidad de los desarrolladores ❤️❤️❤️
* Facilitar un código sencillo, conciso y claro 🤸
* Integración perfecta con Ruby on Rails 🚝

### &#x20;<a href="#new-release-v3.4-developer-happiness-edition" id="new-release-v3.4-developer-happiness-edition"></a>

![](../.gitbook/assets/kittens)

### &#x20;<a href="#faster-uis-smaller-downloads-and-longer-battery-life" id="faster-uis-smaller-downloads-and-longer-battery-life"></a>

Nuestro tamaño de la carga útil de JavaScript sobre la red es un diminuto [**11.4kb** gzipped](https://bundlephobia.com/result?p=stimulus\_reflex@3.4.0)... y eso _incluye_ StimulusReflex, ActionCable, Morphdom y CableReady.

Mientras que StimulusReflex es un enfoque radicalmente diferente que hace difícil hacer una comparación directa con los frameworks SPA populares, la única cosa en la que todo el mundo parece estar de acuerdo es en lo pequeña que es su implementación de la Lista de Todo / TODO List. Aquí están los números:

No todo el mundo tiene el último iPhone en su bolsillo. Entregamos el HTML al cliente, que todos los dispositivos pueden mostrar sin que un marco de trabajo renderice una interfaz de usuario a partir de JSON. Reducimos la complejidad para los desarrolladores al tiempo que facilitamos el acceso a tu sitio a personas con conexiones más lentas y dispositivos menos potentes sin agotar su batería.

### &#x20;<a href="#live-demo" id="live-demo"></a>

Algunas de nuestras demostraciones favoritas son:

* ​[Tabular](https://expo.stimulusreflex.com/demos/tabular): filtrado, ordenación y paginación sin necesidad de JavaScript del cliente
* ​[Todo](https://expo.stimulusreflex.com/demos/todo): nuestra versión del [clásico](https://todomvc.com), con un tamaño de cable entre 2 y 15 veces menor que cualquier otra solución

### &#x20;<a href="#build-the-next-twitter-in-just-9-minutes-or-less" id="build-the-next-twitter-in-just-9-minutes-or-less"></a>

Esta demo de principios de 2020 es emocionante, pero no es un tutorial!.

### &#x20;<a href="#first-class-viewcomponent-support" id="first-class-viewcomponent-support"></a>

Si instalas el increíble [ViewComponentReflex](https://github.com/joshleblanc/view\_component\_reflex), también, podrás persistir el estado de tus componentes en la sesión del usuario. Cada instancia de sus componentes mantendrá su propio estado local. Esto proporciona una continuidad sin fisuras para tu UI - incluso cuando se hace una página completa de Reflexupdates. _Hand, meet glove._ 🖐️+🧤

### &#x20;<a href="#how-we-got-here" id="how-we-got-here"></a>

Nos encanta Rails. Los veteranos del framework recuerdan la sensación de asombro e incredulidad tras ver la obra de David Heinemeier Hansson, [Build a Blog in 15 minutes](bienvenida.md#what-is-stimulusreflex) video. It didn't seem possible that web development could be so easy, productive, and fun. We're talking [exponential gains in developer efficiency](https://www.youtube.com/watch?v=SWEts0rlezA\&t=3m23s) and happiness. Rails has become so successful that nearly every framework since has borrowed ideas, patterns, and features from it.

The landscape has changed a lot since those early days. Applications are more ambitious now. The pursuit of native UI speeds spawned a new breed of increasingly complex technologies. Modern **Single Page Apps** have pushed many of the server's responsibilities to the client. Unfortunately this new approach trades _a developer experience_ that was once **fun and productive** for an alternative of high complexity and only marginal gains.

**There must be a better way.**

### &#x20;<a href="#the-revolution-begins" id="the-revolution-begins"></a>

In his 2018 ElixirConf keynote, [Chris McCord](https://twitter.com/chris\_mccord) _(creator of the_ [_Phoenix_](http://www.phoenixframework.org) _framework for_ [_Elixir_](https://elixir-lang.org)_)_ introduced [LiveView](https://github.com/phoenixframework/phoenix\_live\_view), an alternative to the SPA. His [presentation](https://www.youtube.com/watch?v=8xJzHq8ru0M) captures the same promise and excitement that Rails had in the early days.

We love Elixir and Phoenix. Elixir hits a sweet spot for people who want Rails-like conventions in a functional language. The community is terrific, but it's still small and comparatively niche.

Also, we just really enjoy using **Ruby and Rails**.

StimulusReflex was originally inspired by LiveView, but we are charting our own course. Our goal has always been to make building modern apps with Rails the most productive and enjoyable option available. We want to inspire our friends working with other tools and technologies to evaluate how concepts like StimulusReflex could work in their ecosystems and communities.

So far, it's working! Not only do we now have 20+ developers actively contributing to StimulusReflex, but we've inspired projects like [SockPuppet](https://github.com/jonathan-s/django-sockpuppet) for **Django**.

We are truly stronger together.
