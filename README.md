# node-js-project-on-APIs-to-perform-CRUD-operations-on-the-tables-movie-director

const express = require("express");
const path = require("path");

const { open } = require("sqlite");
const sqlite3 = require("sqlite3");

const app = express();
app.use(express.json());

const dbPath = path.join(__dirname, "moviesData.db");
let db = null;

const initializeDBAndServer = async () => {
  try {
    db = await open({
      filename: dbPath,
      driver: sqlite3.Database,
    });
    app.listen(3000, () => {
      console.log("Server Running at http://localhost:3000/");
    });
  } catch (e) {
    console.log(`DB Error: ${e.message}`);
    process.exit(1);
  }
};
initializeDBAndServer();

const convertMovieNameToPascal = (dbObject) => {
  return {
    movieName: dbObject.movie_name,
  };
};

// API for GET method

app.get("/movies/", async (request, response) => {
  const getMovieQuery = `
    SELECT
    movie_name
    FROM
    movie;`;
  const moviesArray = await db.all(getMovieQuery);
  response.send(
    moviesArray.map((movieName) => convertMovieNameToPascal(movieName))
  );
});

// API for POST method

app.post("/movies/", async (request, response) => {
  const movieDetails = request.body;
  const { directorId, movieName, leadActor } = movieDetails;
  const addMovieQuery = `
    INSERT INTO
    movie (director_id,movie_name,lead_actor)
    VALUES
    (
     ${directorId},
    '${movieName}',
    '${leadActor}');`;
  const dbResponse = await db.run(addMovieQuery);
  response.send("Movie Successfully Added");
});

const convertDbObjectResponseObject = (dbObject) => {
  return {
    movieId: dbObject.movie_id,
    directorId: dbObject.director_id,
    movieName: dbObject.movie_name,
    leadActor: dbObject.lead_actor,
    directorName: dbObject.director_name,
  };
};

// API for GET method ON SPECIFIED QUERY.

app.get("/movies/:movieId/", async (request, response) => {
  const { movieId } = request.params;
  const getMovieQuery = `
    SELECT
    *
    FROM
    movie
    WHERE
    movie_id= ${movieId};`;
  const movie = await db.get(getMovieQuery);
  response.send(convertDbObjectResponseObject(movie));
});

// API for PUT method

app.put("/movies/:movieId/", async (request, response) => {
  const { movieId } = request.params;
  const movieDetails = request.body;
  const { directorId, movieName, leadActor } = movieDetails;
  const updateMovieDetails = `
    UPDATE
    movie
    SET
    director_id=${directorId},
    movie_name='${movieName}',
    lead_actor='${leadActor}'
    WHERE
    movie_id=${movieId};`;
  await db.run(updateMovieDetails);
  response.send("Movie Details Updated");
});

//API for DELETE method

app.delete("/movies/:movieId/", async (request, response) => {
  const { movieId } = request.params;
  const deleteMovieQuery = `
    DELETE from
    movie
    WHERE
    movie_id=${movieId};`;
  await db.run(deleteMovieQuery);
  response.send("Movie Removed");
});

//API for GET method for DIRECTORS table

app.get("/directors/", async (request, response) => {
  const getDirectorQuery = `
    SELECT 
    *
    FROM
    director;`;
  const directorsArray = await db.all(getDirectorQuery);
  response.send(
    directorsArray.map((directors) => convertDbObjectResponseObject(directors))
  );
});

//API FOR GET MEETHOD FOR DIRECTORS ON SPECIFIED QUERY USING INNER JOINT

app.get("/directors/:directorId/movies/", async (request, response) => {
  const { directorId } = request.params;
  const getDirectorQuery = `
    SELECT
    *
    FROM
    director INNER JOIN movie 
    ON director.director_id = movie.movie_id
    WHERE
    director.director_id=${directorId};`;
  const movies = await db.run(getDirectorQuery);
  response.send(
      movies.map((movieNames)=>convertDbObjectResponseObject(movieNames));
});
module.exports = app;
