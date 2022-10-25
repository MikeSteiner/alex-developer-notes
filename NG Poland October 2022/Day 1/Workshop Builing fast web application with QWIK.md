
-   [slides](https://docs.google.com/presentation/d/1UNKyLC-KJsYcRHS813bi0bv6-f1Sv1nczIOOxh11y8A/edit)
-   [Streaming demo](https://qwik-multi-worker-main.devdash.workers.dev/)
-   [https://qwik-storefront.vendure.io/](https://qwik-storefront.vendure.io/)
-   [micro-frontend personalization demo work in progress](https://qwik-dream-demo.pages.dev/)
-   [https://mitosis.builder.io/](https://mitosis.builder.io/)
-   [https://partytown.builder.io/](https://partytown.builder.io/)
-   [https://qwik.builder.io/](https://qwik.builder.io/)
-   [https://github.com/manucorporat/qwik-conference-app](https://github.com/manucorporat/qwik-conference-app)

```
npm create qwik@latest
```

```
import { component$, useClientEffect$, useStore, $ } from "@builder.io/qwik";

export const step = () => 5;
export default component$(() => {
  const state = useStore({ count: 123, step: 5 });
  return (
    <>
      <span>{state.count}</span>
      <button onClick$={async () => (state.count += step())}>+1</button>
      <button onMouseMove$={() => console.log("counter2")}>+2</button>
      <div style={{ height: "500px" }}>scroll</div>
      <Clock />
      <div style={{ height: "50px" }}>space</div>
      <Clock />
    </>
  );
});

export const Clock = component$(() => {
  const state = useStore({ time: "" });
  useClientEffect$(() => {
    const update = () => {
      state.time = new Date().toLocaleTimeString();
    };
    update();
    const interval = setInterval(update, 1000);
    return () => clearInterval(interval);
  });
  return <div>{state.time.toUpperCase()}</div>;
});

//export const SYM = () => state.count++;

```

```
npm add --save tmdb-ts-api
```

```
const tmdb = new Tmdb({ apiKey: "73e6f6e34b28a4c3be0f745176c8225b" });
```

[https://www.themoviedb.org/documentation/api](https://www.themoviedb.org/documentation/api)

```
import { component$, Resource, useStylesScoped$ } from "@builder.io/qwik";
import { RequestHandler, useEndpoint } from "@builder.io/qwik-city";
import { Tmdb } from "tmdb-ts-api";
import { PopularMovies, Result } from "tmdb-ts-api/dist/movie/popular";
import GRID_CSS from "./grid.css?inline";
import POSTER_CSS from "./poster.css?inline";

const tmdb = new Tmdb({ apiKey: "73e6f6e34b28a4c3be0f745176c8225b" });

export const onGet: RequestHandler<PopularMovies> = async () => {
  console.log("get popular movies");
  return await tmdb.movies.getPopular();
};

export default component$(() => {
  useStylesScoped$(GRID_CSS);
  const popularMoviesResource = useEndpoint<PopularMovies>();
  return (
    <div class="grid">
      <Resource
        value={popularMoviesResource}
        onPending={() => <div>Loading...</div>}
        onResolved={(popularMovies) => (
          <>
            {popularMovies.results.map((movie) => (
              <MoviePoster movie={movie} />
            ))}
          </>
        )}
      />
    </div>
  );
});

export const MoviePoster = component$((props: { movie: Result }) => {
  useStylesScoped$(POSTER_CSS);
  const rating = props.movie.vote_average;
  return (
    <div class="poster">
      <div class="title">{props.movie.title}</div>
      <div>{props.movie.genre_ids}</div>
      <div>
        <img
          src={"https://image.tmdb.org/t/p/w342/" + props.movie.poster_path}
        />
      </div>
      <div class="rating">
        <div class="bar" onClick$={() => console.log(rating)}>
          <div
            class="front"
            style={{ width: props.movie.vote_average / 2 + "em" }}
          >
            ★★★★★
          </div>
          <div class="back">★★★★★</div>
        </div>
      </div>
    </div>
  );
});
```

grid

```
.poster {
  width: 200px;
  margin: 1em;
  text-align: center;
}

img {
  width: 100%;
}

.title {
  font-size: 1.5em;
  font-weight: bold;
}

.rating {
  font-size: 2em;
  font-family: "Montserrat", sans-serif;
  font-weight: 400;
}

.rating .back {
  /* position: absolute;*/
  color: lightgray;
}

.bar {
  display: inline-block;
}

.rating .front {
  position: absolute;
  width: 3em;
  overflow: hidden;
  z-index: 1;
}

```

[https://qwik-dream-demo.pages.dev/](https://qwik-dream-demo.pages.dev/)
