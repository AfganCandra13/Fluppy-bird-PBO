import javafx.animation.KeyFrame;
import javafx.animation.Timeline;
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.input.KeyCode;
import javafx.scene.layout.Pane;
import javafx.scene.paint.Color;
import javafx.scene.shape.Rectangle;
import javafx.scene.text.Text;
import javafx.stage.Stage;
import javafx.util.Duration;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

// Abstract Class for Game Objects
abstract class GameObject {
    protected double x, y;

    public GameObject(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public abstract void update();

    public abstract void draw(Pane pane);

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}

// Bird Class
class Bird extends GameObject {
    private double velocity;
    private static final double GRAVITY = 0.5;
    private static final double FLAP_STRENGTH = -10;
    private final Rectangle shape;

    public Bird(double x, double y) {
        super(x, y);
        this.velocity = 0;
        shape = new Rectangle(20, 20, Color.YELLOW);
        shape.setX(x);
        shape.setY(y);
    }

    @Override
    public void update() {
        velocity += GRAVITY;
        y += velocity;
        shape.setY(y);
    }

    @Override
    public void draw(Pane pane) {
        if (!pane.getChildren().contains(shape)) {
            pane.getChildren().add(shape);
        }
    }

    public void flap() {
        velocity = FLAP_STRENGTH;
    }

    public boolean isOutOfBounds(double height) {
        return y > height || y < 0;
    }

    public Rectangle getShape() {
        return shape;
    }
}

// Pipe Class
class Pipe extends GameObject {
    private static final double WIDTH = 50;
    private final Rectangle topPipe;
    private final Rectangle bottomPipe;
    private static final double SPEED = 3;

    public Pipe(double x, double gapY, double gapHeight, double screenHeight) {
        super(x, 0);
        topPipe = new Rectangle(WIDTH, gapY, Color.GREEN);
        bottomPipe = new Rectangle(WIDTH, screenHeight - gapY - gapHeight, Color.GREEN);
        bottomPipe.setY(gapY + gapHeight);
        topPipe.setX(x);
        bottomPipe.setX(x);
    }

    @Override
    public void update() {
        x -= SPEED;
        topPipe.setX(x);
        bottomPipe.setX(x);
    }

    @Override
    public void draw(Pane pane) {
        if (!pane.getChildren().contains(topPipe)) {
            pane.getChildren().add(topPipe);
            pane.getChildren().add(bottomPipe);
        }
    }

    public boolean isOffScreen() {
        return x + WIDTH < 0;
    }

    public boolean collidesWith(Bird bird) {
        return bird.getShape().getBoundsInParent().intersects(topPipe.getBoundsInParent()) ||
               bird.getShape().getBoundsInParent().intersects(bottomPipe.getBoundsInParent());
    }
}

// Main Game Class
public class FlappyBirdGame extends Application {
    private static final double WIDTH = 600;
    private static final double HEIGHT = 400;
    private List<Pipe> pipes = new ArrayList<>();
    private Bird bird;
    private Pane pane;
    private static int score = 0;
    private Text scoreText;
    private Timeline gameLoop;
    private boolean gameStarted = false;

    public static void main(String[] args) {
        launch(args);
    }

    public static int getScore() {
        return score;
    }

    public static void setScore(int newScore) {
        score = newScore;
    }

    @Override
    public void start(Stage primaryStage) {
        pane = new Pane();
        Scene scene = new Scene(pane, WIDTH, HEIGHT);
        scene.setFill(Color.CYAN);

        bird = new Bird(100, HEIGHT / 2);
        bird.draw(pane);

        scoreText = new Text(10, 20, "Score: 0");
        scoreText.setFill(Color.BLACK);
        pane.getChildren().add(scoreText);

        Text startText = new Text(WIDTH / 2 - 100, HEIGHT / 2, "Tekan Enter untuk memulai");
        startText.setFill(Color.BLACK);
        pane.getChildren().add(startText);

        scene.setOnKeyPressed(event -> {
            if (event.getCode() == KeyCode.SPACE) {
                if (!gameStarted) {
                    startText.setVisible(false);
                    startGame();
                }
                bird.flap();
            }
        });

        primaryStage.setTitle("Flappy Bird Game");
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    private void startGame() {
        gameStarted = true;

        gameLoop = new Timeline(new KeyFrame(Duration.millis(20), e -> {
            try {
                update();
            } catch (Exception ex) {
                gameOver();
            }
        }));
        gameLoop.setCycleCount(Timeline.INDEFINITE);
        gameLoop.play();

        Timeline pipeSpawner = new Timeline(new KeyFrame(Duration.seconds(2), e -> spawnPipes()));
        pipeSpawner.setCycleCount(Timeline.INDEFINITE);
        pipeSpawner.play();
    }

    private void update() throws Exception {
        bird.update();
        if (bird.isOutOfBounds(HEIGHT)) {
            throw new Exception("Bird Out of Bounds");
        }

        List<Pipe> pipesToRemove = new ArrayList<>();
        for (Pipe pipe : pipes) {
            pipe.update();
            pipe.draw(pane);
            if (pipe.collidesWith(bird)) {
                throw new Exception("Collision Detected");
            }
            if (pipe.isOffScreen()) {
                pipesToRemove.add(pipe);
                score++;
                scoreText.setText("Score: " + score);
            }
        }
        pipes.removeAll(pipesToRemove);
    }

    private void spawnPipes() {
        Random random = new Random();
        double gapY = random.nextInt((int) (HEIGHT - 150));
        Pipe pipe = new Pipe(WIDTH, gapY, 150, HEIGHT);
        pipes.add(pipe);
    }

    private void gameOver() {
        gameLoop.stop();
        Text gameOverText = new Text(WIDTH / 2 - 100, HEIGHT / 2, "Game Over! Final Score: " + score);
        gameOverText.setFill(Color.BLACK);
        pane.getChildren().add(gameOverText);
    }
}
