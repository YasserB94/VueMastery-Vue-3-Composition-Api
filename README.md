# vue-3-composition-api

This project contains my code and notes following along's Vue Mastery's Vue 3 Composition Api tutorial

## Why the composition API ?

- More readable
- Easier reusable code
- Easier to maintain
- Less configuration
- More performant when reusing code with composition functions
- Better intellisense

---

## When to use the Composition API

Note: the composition API is an addition to vue, any code written in the older syntax (options api) is still completelky valid
Reasons to use the composition API

- TypeScript Support.
- Organized code
- Reusable code
- Your team prefers it

> _Or in Short IMO: If you care, at all_

---

## Setup & Reactive preferences

### Setup Method

#### Executes before:

- Components
- Props
- Data
- Methods
- Computed Properties
- Lifecycle methods

#### Has 2 Optional arguments : Props and Context

The context and the props can be watched

#### 1.Props

```js
import { watch } from "vue";
export default {
  //Declare props
  props: {
    greeting: String,
  },
  //Pas it into the setup method
  setup(props) {
    //Watch it, and log it when it changes
    watch(() => {
      console.log(props.greeting);
    });
  },
};
```

#### 2.Context

We can specify a component's context by passing in a second argument
The context can be accessed with: `this`

##### It gives us access to:

```js
export default {
  setup(props, context) {
    context.attrs;
    context.slots;
    context.parent;
    context.root;
    context.emit;
  },
};
```

### Reactive methods

```html
<template>
  <p>Count: {{count}}</p>
</template>
<script>
  import { ref } from "vue";
  export default {
    setup() {
      const count = ref(4);
      //The setup method has to return anything our template needs, in this case the 'count' variable
      return { count };
    },
  };
</script>
```

##### Ref

- Creates a **Reactive Reference**
- Wraps our primitive in an Object
  - Which allows us to track changes to it
- In the options API this was done within data()
- Note:
  - With the composition api we can declare reactive objects that aren't associated with a component!

---

## Methods

How do we add methods in the composition api ?

```html
<template>
  <p>Count: {{count}}</p>
  <!--Let's add a button that incrememts on click-->
  <button @click="incrementCount()">Increment</button>
</template>
<script>
  import { ref } from "vue";
  export default {
    setup() {
      const count = ref(4);
      //We can declare normal functions inside setup
      function incrementCount() {
        //Since count is a 'ref' object, we increment it's value
        count.value++;
        //Hot take
        //Vue automatically renders the value of any reactive object in the template
      }
      //And make it accessible to our template by returning the function
      return { count, incrementCount };
    },
  };
</script>
```

## Computed Properties

Computed properties, allow us to take a lot of logic out of our templates(views) as we can 'compute' a value(and use reactive objects) and make it accessible to the view

```html
<template>
  <p>Count: {{count}}</p>
  <button @click="incrementCount()">Increment</button>
  <p>Count the names!</p>
  <ul>
    <li v-for="name in names" :key="name">{{name}}</li>
  </ul>
  <!--Whenever the user clicks we'll see -->
  <p>Left to count: {{countingRemaining}}</p>
</template>
<script>
  import { ref, computed } from "vue";
  export default {
    setup() {
      const count = ref(0);
      const names = ref(["Tim", "Bob", "Joe"]);
      function incrementCount() {
        count.value++;
      }
      function countingRemaining = computed(()=>{

        return names.value.length - count.value
      })
      return { count, incrementCount, countingRemaining };
    },
  };
</script>
```

---

## Reactive Syntax

In the composition api next to **ref** we can also use **reactive** to declare reactive properties.
Reactive allows us to return a reactive **object** instead of a primitive.

```html
<template>
  <p>There are {{event.ticketsLeft}} tickets left</p>
</template>
<script>
  import  {reactive,computed} from "vue";
  export default {
      setup(){
          const event = reactive({
             name:"Tommorowland",
             capacity:245_000,
             ticketsSold:148_000,
             //Note we can use computed within the object's properties.
             //We no longer need .value since our object takes care of reactivity
             ticketsLeft: computed()=>{
                return event.capacity - event.ticketsSold;
             }
          })
          //We can just return our reactive object ot be accessible by the view(template)
      return { event };
      }
    }
</script>
```

### toRefs

toRefs converts a reactive object to plain objects where each property becomes a reactive **Ref**erence pointing towards the property on the original object

```html
<template>
  <p>There are {{ticketsLeft}} tickets left</p>
</template>
<script>
      import  {reactive,computed,toRefs} from "vue";
  export default {
      setup(){
          const event = reactive({
             name:"Tommorowland",
             capacity:245_000,
             ticketsSold:148_000,
             ticketsLeft: computed()=>{
                return event.capacity - event.ticketsSold;
             }
          })
          //Every property on the object becomes a Ref.
      return { ...toRefs(event) };
      }
    }
</script>
```

---

## Modularizing

We can make code more organized and readable by using composition functions in Vue3

```html
<template> ... </template>
<script>
  import { ref, computed } from "vue";
  export default {
    setup() {
      return useEventSpace(); //This function was extracted form Setup
    },
  };
  function useEventSpace() {
    const capacity = ref(4);
    const attending = ref(["Tim", "Bob", "Joe"]);
    const spacesLeft = computed(() => {
      return capacity.value - attending.value.length;
    });
    function increaseCapacity() {
      capacity.value++;
    }
    return { capacity, attending, spacesLeft, increaseCapacity };
  }
</script>
```

Now We move the function to a seperate file, so it's reusable across the project

```js
import { ref, computed } from "vue";

export default function useEventSpace() {
  const capacity = ref(4);
  const attending = ref(["Tim", "Bob", "Joe"]);
  const spacesLeft = computed(() => {
    return capacity.value - attending.value.length;
  });
  function increaseCapacity() {
    capacity.value++;
  }
  return { capacity, attending, spacesLeft, increaseCapacity };
}
```

And we import it into the component to 'use' it

```html
<template> ... </template>
<script>
  import useEventSpace from "@/use/event-space";
  export default {
    setup() {
      return useEventSpace();
    },
  };
</script>
```

**!!!-BEST PRACTICE-!!!**

When using a lot of composition functions or composoables for short, We create a problem. What composable provides what data ? (which was the problem with the options api's "mixins" in vue2)
We can solve the problem by using local objects within our component.

```html
<template> ... </template>
<script>
  import useEventSpace from "@/use/event-space";
  import useMapping from "@/use/mapping";
  export default {
    setup() {
      const { capacity, attending, spacesLeft, increaseCapacity } =
        useEventSpace();
      const { map, embedId } = useMapping();

      return {
        capacity,
        attending,
        spacesLeft,
        increaseCapacity,
        map,
        embedId,
      };
    },
  };
</script>
```

---

## LifeCycle Hooks

Vue provides a bunch of lifecycle hooks.
Their names differ from the Vue2 names, since the creator of Vue admitted his initial naming,well.... Just wasn't good.

- Vue 3 LifeCycle Methods ( - Composition API)
  - onMounted()
  - onUpdated()
  - onUnmounted()
  - onBeforeMount()
  - onBeforeUpdate()
  - onBeforeUnmount()
  - onErrorCaptured()
  - onRenderTracked() -- NEW (debugging,optimization)
  - onRenderTriggered() -- NEW (debugging,optimization)
  - onActivated()
  - onDeactivated()
  - onServerPrefetch()

---

## Watch

Explained with code

### WatchEffect

The problem->

```html
<template>
  <div>
    <!---The user searches for events-->
    Search for <input v-model="searchInput" />
    <div>
      <!---We show the amount of events found-->
      <p>Number of events: {{ results }}</p>
    </div>
  </div>
</template>
<script>
  import { ref } from "@vue/composition-api";
  //Mock API
  import eventApi from "@/api/event.js";

  export default {
    setup() {
      //Track the search input as reactive value
      const searchInput = ref("");
      //Make sure our feedback amount is reactive
      const results = ref(0);
      //We  call the api and update the value
      results.value = eventApi.getEventCount(searchInput.value);
      //Send both to our view
      return { searchInput, results };
    },
  };
</script>
```

The Setup function only runs whenever the component is first created, Therefor changing the reactive values (refs) run everything, but don't call the API with the new values
**The Solution: WatchEffect()**

```js
setup() {
  const searchInput = ref("");
  const results = ref(0);

  watchEffect(
    //On setup watcheffect will register all our reactive variables as dependencies.
    //Everytime any of the refs change, the callback function will trigger once more
    () => {
    results.value = eventApi.getEventCount(searchInput.value);
  });

  return { searchInput, results };
}
```

Tough this results into another problem, Since watchEffect automaticly detects it's dependencies. To declare the dependencies ourselves. We use **Watch**

### Watch

```js
//Only trigger the callback when a specified value change
watch(searchInput, () => {
  ...
});
//Give me the new and old value within my callback function
watch(searchInput, (newVal, oldVal) => {
  ...
});
//Run the callback tracking more dependencies
watch([firstName, lastName], () => {
  ...  
});
//Run the callback tracking more dependencies and providing the old/new values
watch([firstName, lastName], ([newFirst, newLast], [oldFirst, oldLast]) => {
  ...   
});
```

## Suspense

Suspense is a built in Component that allows us to wrap 2 templates.
This is handy to for exa,ple provide feedback to the user whenever we are awaiting an API call.

```html
<template>
  <Suspense>
    <template #default>
      <!-- Put component/components here, one or more of which makes an asychronous call -->
    </template>
    <template #fallback>
      <!-- What to display when loading -->
    </template>
  </Suspense>
</template>
<!---Or with content:-->
<template>
  <Suspense>
    <template #default>
      <Event />
    </template>
    <template #fallback> Loading... </template>
  </Suspense>
</template>
<script>
  import Event from "@/components/Event.vue";
  export default {
    components: { Event },
  };
</script>
```

### Deeply Nested Async Calls

What’s even more powerful is that I might have a deeply nested component that has an asynchronous call. Suspense will wait for all asynchronous calls to finish before loading the template. So you can have one loading screen on your app, that waits for multiple parts of your application to load.

---

## Teleport

there are some instances where one component has some html that needs to get rendered in an alternative location. For example:

- Styles that require fixed or absolute positioning and z-index. For example, it’s a common pattern to place UI components (like modals) right before the </body> tag to ensure they are properly placed in front of all other parts of the webpage.
- When our Vue application is running on a small part of our webpage (or a widget), sometimes we may want to move components to other locations in the DOM outside of our Vue app.

```html
<template>
  <teleport to="#some-id">
    <div>
      <p>I will be teleported to the element with id #some-id</p>
    </div>
  </teleport>
  <teleport to=".someClass">
    <p>I will be teleported to the element with class .someClass</p>
  </teleport>
</template>
```

Component's state is preserved when teleporting!
