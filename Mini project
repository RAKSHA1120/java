import java.sql.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class MovieTicketBookingSystem {

    // Database connection details (Assumed to be running locally)
    private static final String DB_URL = "jdbc:mysql://localhost:3306/movie_db";
    private static final String USER = "root";
    private static final String PASSWORD = "password";

    public static void main(String[] args) {
        // Start the movie ticket booking process
        Scanner scanner = new Scanner(System.in);

        // Fetch movies from the database
        try {
            List<Movie> movies = getAllMovies();
            System.out.println("Movies Available:");
            for (int i = 0; i < movies.size(); i++) {
                System.out.println((i + 1) + ". " + movies.get(i));
            }

            System.out.print("Enter movie number to view showtimes: ");
            int movieChoice = scanner.nextInt();
            Movie selectedMovie = movies.get(movieChoice - 1);

            // Fetch and display showtimes for the selected movie
            List<Showtime> showtimes = getShowtimesForMovie(selectedMovie.getMovieId());
            System.out.println("Showtimes for " + selectedMovie.getTitle() + ":");
            for (int i = 0; i < showtimes.size(); i++) {
                System.out.println((i + 1) + ". " + showtimes.get(i));
            }

            System.out.print("Enter showtime number to book tickets: ");
            int showtimeChoice = scanner.nextInt();
            Showtime selectedShowtime = showtimes.get(showtimeChoice - 1);

            // Display seat availability
            System.out.println("Seats available for this showtime:");
            boolean[] seats = selectedShowtime.getSeats();
            for (int i = 0; i < seats.length; i++) {
                System.out.println("Seat " + (i + 1) + (seats[i] ? " (Booked)" : " (Available)"));
            }

            System.out.print("Enter seat numbers to book (comma separated): ");
            scanner.nextLine();  // Consume newline left by nextInt()
            String seatInput = scanner.nextLine();
            String[] seatNumbers = seatInput.split(",");
            List<Integer> seatList = new ArrayList<>();
            for (String seat : seatNumbers) {
                seatList.add(Integer.parseInt(seat.trim()) - 1);  // Convert to zero-based index
            }

            // Confirm booking
            Booking booking = new Booking(selectedShowtime, seatList);
            try {
                booking.confirmBooking();
                System.out.println("Booking confirmed!");
            } catch (IllegalStateException e) {
                System.out.println("Error: " + e.getMessage());
            }

        } catch (SQLException e) {
            System.out.println("Error connecting to database: " + e.getMessage());
        }
    }

    // Movie class representing a movie
    static class Movie {
        private int movieId;
        private String title;
        private String genre;
        private String description;

        public Movie(int movieId, String title, String genre, String description) {
            this.movieId = movieId;
            this.title = title;
            this.genre = genre;
            this.description = description;
        }

        public int getMovieId() {
            return movieId;
        }

        public String getTitle() {
            return title;
        }

        public String getGenre() {
            return genre;
        }

        public String getDescription() {
            return description;
        }

        @Override
        public String toString() {
            return title + " (" + genre + ")";
        }
    }

    // Showtime class representing a movie's showtime
    static class Showtime {
        private int showtimeId;
        private Movie movie;
        private LocalDateTime showTime;
        private boolean[] seats; // true = booked, false = available

        public Showtime(int showtimeId, Movie movie, LocalDateTime showTime, int totalSeats) {
            this.showtimeId = showtimeId;
            this.movie = movie;
            this.showTime = showTime;
            this.seats = new boolean[totalSeats];
        }

        public int getShowtimeId() {
            return showtimeId;
        }

        public Movie getMovie() {
            return movie;
        }

        public LocalDateTime getShowTime() {
            return showTime;
        }

        public boolean[] getSeats() {
            return seats;
        }

        public boolean isSeatAvailable(int seatIndex) {
            return !seats[seatIndex];
        }

        public void bookSeat(int seatIndex) throws IllegalStateException {
            if (seats[seatIndex]) {
                throw new IllegalStateException("Seat " + (seatIndex + 1) + " is already booked!");
            }
            seats[seatIndex] = true;
        }

        @Override
        public String toString() {
            return movie.getTitle() + " - " + showTime.toString();
        }
    }

    // Booking class representing a user's booking
    static class Booking {
        private Showtime showtime;
        private List<Integer> seatNumbers; // List of seat indexes

        public Booking(Showtime showtime, List<Integer> seatNumbers) {
            this.showtime = showtime;
            this.seatNumbers = seatNumbers;
        }

        public void confirmBooking() throws IllegalStateException {
            for (int seatIndex : seatNumbers) {
                if (!showtime.isSeatAvailable(seatIndex)) {
                    throw new IllegalStateException("One or more seats are already booked.");
                }
                showtime.bookSeat(seatIndex);  // Book the seat
            }
        }
    }

    // Database handling class for fetching movie and showtime data using JDBC
    static class MovieDB {

        // Get all movies from the database
        public static List<Movie> getAllMovies() throws SQLException {
            List<Movie> movies = new ArrayList<>();
            String query = "SELECT * FROM movies";
            try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
                 Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery(query)) {

                while (rs.next()) {
                    int movieId = rs.getInt("movie_id");
                    String title = rs.getString("title");
                    String genre = rs.getString("genre");
                    String description = rs.getString("description");
                    movies.add(new Movie(movieId, title, genre, description));
                }
            }
            return movies;
        }

        // Get showtimes for a specific movie
        public static List<Showtime> getShowtimesForMovie(int movieId) throws SQLException {
            List<Showtime> showtimes = new ArrayList<>();
            String query = "SELECT * FROM showtimes WHERE movie_id = ?";
            try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
                 PreparedStatement pstmt = conn.prepareStatement(query)) {
                pstmt.setInt(1, movieId);
                try (ResultSet rs = pstmt.executeQuery()) {
                    while (rs.next()) {
                        int showtimeId = rs.getInt("showtime_id");
                        LocalDateTime showTime = rs.getTimestamp("showtime").toLocalDateTime();
                        int totalSeats = rs.getInt("total_seats");
                        Movie movie = getMovieById(movieId);
                        showtimes.add(new Showtime(showtimeId, movie, showTime, totalSeats));
                    }
                }
            }
            return showtimes;
        }

        // Get a movie by its ID
        private static Movie getMovieById(int movieId) throws SQLException {
            String query = "SELECT * FROM movies WHERE movie_id = ?";
            try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
                 PreparedStatement pstmt = conn.prepareStatement(query)) {
                pstmt.setInt(1, movieId);
                try (ResultSet rs = pstmt.executeQuery()) {
                    if (rs.next()) {
                        String title = rs.getString("title");
                        String genre = rs.getString("genre");
                        String description = rs.getString("description");
                        return new Movie(movieId, title, genre, description);
                    }
                }
            }
            return null;
        }
    }
}
