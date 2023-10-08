import com.badlogic.gdx.ApplicationAdapter;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input.Keys;
import com.badlogic.gdx.graphics.Color;
import com.badlogic.gdx.graphics.GL20;
import com.badlogic.gdx.graphics.g2d.BitmapFont;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Circle;
import com.badlogic.gdx.math.Vector2;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class AvoidTheEnemiesGame extends ApplicationAdapter {
    private static final int WIDTH = 800;
    private static final int HEIGHT = 600;

    private SpriteBatch batch;
    private BitmapFont font;
    private Circle player;
    private List<Enemy> enemies;
    private int score = 0;
    private int highScore;
    private boolean gameOver = false;
    private float gameOverDuration = 0;

    private float enemySpawnTimer = 0;
    private float speedIncreaseTimer = 0;
    private float spawnInterval = 10; // Spawn a new enemy every 10 seconds
    private float speedIncreaseInterval = 15; // Increase enemy speed every 15 seconds

    private Random random = new Random();

    @Override
    public void create() {
        batch = new SpriteBatch();
        font = new BitmapFont();

        player = new Circle(WIDTH / 2f, HEIGHT / 2f, 20f);
        enemies = new ArrayList<Enemy>();

        highScore = loadHighScore();
    }

    @Override
    public void render() {
        handleInput();

        if (!gameOver) {
            movePlayer();
            checkCollisions();

            for (Enemy enemy : enemies) {
                enemy.move();
                enemy.checkBounds();
                if (enemy.isColliding(player)) {
                    gameOver = true;
                    gameOverDuration = 0;
                }
            }

            score++;
            spawnEnemies();
            increaseEnemySpeed();
        }

        updateHighScore();

        // Clear the screen
        Gdx.gl.glClearColor(0, 0, 0, 1);
        Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);

        batch.begin();

        // Draw player
        batch.setColor(Color.RED);
        batch.draw(getCircleTexture(20), player.x - player.radius, player.y - player.radius);

        // Draw enemies
        batch.setColor(Color.GREEN);
        for (Enemy enemy : enemies) {
            batch.draw(getCircleTexture(15), enemy.getPosition().x - enemy.getRadius(), enemy.getPosition().y - enemy.getRadius());
        }

        // Draw score and high score
        font.draw(batch, "Score: " + score, 10, HEIGHT - 10);
        font.draw(batch, "High Score: " + highScore, 10, HEIGHT - 50);

        // Draw game over message
        if (gameOver) {
            font.getData().setScale(2);
            font.draw(batch, "Game Over", WIDTH / 2 - 120, HEIGHT / 2 + 50);
            font.getData().setScale(1);
            font.draw(batch, "Press Enter To Respawn", WIDTH / 2 - 120, HEIGHT / 2 - 50);
        }

        batch.end();
    }

    @Override
    public void dispose() {
        batch.dispose();
        font.dispose();
    }

    private void handleInput() {
        if (gameOver) {
            if (Gdx.input.isKeyPressed(Keys.ENTER)) {
                restartGame();
            }
        }
    }

    private void movePlayer() {
        if (Gdx.input.isKeyPressed(Keys.LEFT) && player.x > player.radius) {
            player.x -= 5;
        }
        if (Gdx.input.isKeyPressed(Keys.RIGHT) && player.x < WIDTH - player.radius) {
            player.x += 5;
        }
        if (Gdx.input.isKeyPressed(Keys.UP) && player.y > player.radius) {
            player.y -= 5;
        }
        if (Gdx.input.isKeyPressed(Keys.DOWN) && player.y < HEIGHT - player.radius) {
            player.y += 5;
        }
    }

    private void checkCollisions() {
        for (Enemy enemy : enemies) {
            if (enemy.isColliding(player)) {
                gameOver = true;
                gameOverDuration = 0;
                break;
            }
        }
    }

    private void spawnEnemies() {
        if (TimeUtils.timeSinceMillis(enemySpawnTimer) > spawnInterval * 1000) {
            spawnEnemy();
            enemySpawnTimer = TimeUtils.millis();
        }
    }

    private void spawnEnemy() {
        float enemyX, enemyY;
        do {
            enemyX = random.nextFloat() * (WIDTH - 2 * enemyRadius) + enemyRadius;
            enemyY = random.nextFloat() * (HEIGHT - 2 * enemyRadius) + enemyRadius;
        } while (player.dst(enemyX, enemyY) < player.radius + enemyRadius + 25);

        float enemySpeed = 2;
        enemies.add(new Enemy(new Vector2(enemyX, enemyY), enemySpeed));
    }

    private void increaseEnemySpeed() {
        if (TimeUtils.timeSinceMillis(speedIncreaseTimer) > speedIncreaseInterval * 1000) {
            for (Enemy enemy : enemies) {
                enemy.increaseSpeed();
            }
            speedIncreaseTimer = TimeUtils.millis();
        }
    }

    private void updateHighScore() {
        if (score > highScore) {
            highScore = score;
            saveHighScore(highScore);
        }
    }

    private void restartGame() {
        player.setPosition(WIDTH / 2f, HEIGHT / 2f);
        enemies.clear();
        score = 0;
        gameOver = false;
    }

    private int loadHighScore() {
        Preferences preferences = Gdx.app.getPreferences("HighScore");
        return preferences.getInteger("highScore", 0);
    }

    private void saveHighScore(int score) {
        Preferences preferences = Gdx.app.getPreferences("HighScore");
        preferences.putInteger("highScore", score);
        preferences.flush();
    }

    private Texture getCircleTexture(int radius) {
        Pixmap pixmap = new Pixmap(radius * 2, radius * 2, Pixmap.Format.RGBA8888);
        pixmap.setColor(1, 1, 1, 0);
        pixmap.fill();
        pixmap.setColor(1, 1, 1, 1);
        pixmap.fillCircle(radius, radius, radius);
        Texture texture = new Texture(pixmap);
        pixmap.dispose();
        return texture;
    }
}
