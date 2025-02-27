# CINTHIA-Y-GUS-Amor-en-linea-1

package com.example.romanticshooter;

import com.badlogic.gdx.ApplicationAdapter;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input.Keys;
import com.badlogic.gdx.graphics.GL20;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.math.Intersector;
import com.badlogic.gdx.utils.Array;

public class RomanticShooterGame extends ApplicationAdapter {
    SpriteBatch batch;
    Texture playerTexture, enemyTexture, heartTexture, bulletTexture;
    Rectangle player;
    Array<Rectangle> enemies, hearts, bullets;
    float enemySpawnTimer, heartSpawnTimer, bulletSpawnTimer;
    int score;
    
    @Override
    public void create () {
        batch = new SpriteBatch();
        // Recuerda incluir las imágenes en tu carpeta assets
        playerTexture = new Texture("player.png");
        enemyTexture = new Texture("enemy.png");
        heartTexture = new Texture("heart.png");
        bulletTexture = new Texture("bullet.png");
        
        // Configuración inicial del jugador
        player = new Rectangle();
        player.x = Gdx.graphics.getWidth() / 2 - 32;
        player.y = 20;
        player.width = 64;
        player.height = 64;
        
        enemies = new Array<>();
        hearts = new Array<>();
        bullets = new Array<>();
        enemySpawnTimer = 0;
        heartSpawnTimer = 0;
        bulletSpawnTimer = 0;
        score = 0;
    }

    @Override
    public void render () {
        // Limpiamos la pantalla y pintamos el fondo (un toque oscuro para resaltar la acción)
        Gdx.gl.glClearColor(0, 0, 0.2f, 1);
        Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);
        
        float deltaTime = Gdx.graphics.getDeltaTime();
        updateGame(deltaTime);
        
        batch.begin();
        batch.draw(playerTexture, player.x, player.y);
        for (Rectangle enemy : enemies)
            batch.draw(enemyTexture, enemy.x, enemy.y);
        for (Rectangle heart : hearts)
            batch.draw(heartTexture, heart.x, heart.y);
        for (Rectangle bullet : bullets)
            batch.draw(bulletTexture, bullet.x, bullet.y);
        // Aquí podrías agregar un BitmapFont para mostrar el puntaje y un mensaje romántico “level up”
        batch.end();
    }
    
    public void updateGame(float deltaTime) {
        // Generamos enemigos cada 1 segundo
        enemySpawnTimer += deltaTime;
        if (enemySpawnTimer > 1) {
            Rectangle enemy = new Rectangle();
            enemy.x = (float) Math.random() * (Gdx.graphics.getWidth() - 64);
            enemy.y = Gdx.graphics.getHeight();
            enemy.width = 64;
            enemy.height = 64;
            enemies.add(enemy);
            enemySpawnTimer = 0;
        }
        
        // Generamos power-ups (corazones) cada 5 segundos
        heartSpawnTimer += deltaTime;
        if (heartSpawnTimer > 5) {
            Rectangle heart = new Rectangle();
            heart.x = (float) Math.random() * (Gdx.graphics.getWidth() - 32);
            heart.y = Gdx.graphics.getHeight();
            heart.width = 32;
            heart.height = 32;
            hearts.add(heart);
            heartSpawnTimer = 0;
        }
        
        // Movemos enemigos y corazones hacia abajo
        for (Rectangle enemy : enemies) {
            enemy.y -= 200 * deltaTime;
        }
        for (Rectangle heart : hearts) {
            heart.y -= 150 * deltaTime;
        }
        
        // Movimiento del jugador (toca en la pantalla para mover)
        if (Gdx.input.isTouched()) {
            float touchX = Gdx.input.getX();
            player.x = touchX - player.width / 2;
        }
        
        // Disparo de balas: presiona la barra espaciadora (ideal para pruebas en escritorio)
        bulletSpawnTimer += deltaTime;
        if (Gdx.input.isKeyPressed(Keys.SPACE) && bulletSpawnTimer > 0.3f) {
            Rectangle bullet = new Rectangle();
            bullet.x = player.x + player.width / 2 - 8;
            bullet.y = player.y + player.height;
            bullet.width = 16;
            bullet.height = 32;
            bullets.add(bullet);
            bulletSpawnTimer = 0;
        }
        
        // Mover balas hacia arriba
        for (Rectangle bullet : bullets) {
            bullet.y += 300 * deltaTime;
        }
        
        // Detectar colisiones: balas contra enemigos
        for (int i = enemies.size - 1; i >= 0; i--) {
            Rectangle enemy = enemies.get(i);
            for (int j = bullets.size - 1; j >= 0; j--) {
                Rectangle bullet = bullets.get(j);
                if (Intersector.overlaps(enemy, bullet)) {
                    enemies.removeIndex(i);
                    bullets.removeIndex(j);
                    score += 10;
                    break;
                }
            }
        }
        
        // Colisión: jugador recoge corazón (aquí se suma “amor extra”)
        for (int i = hearts.size - 1; i >= 0; i--) {
            Rectangle heart = hearts.get(i);
            if (Intersector.overlaps(player, heart)) {
                hearts.removeIndex(i);
                score += 20;
            }
        }
        
        // Remover objetos que salgan de la pantalla
        for (int i = bullets.size - 1; i >= 0; i--) {
            if (bullets.get(i).y > Gdx.graphics.getHeight())
                bullets.removeIndex(i);
        }
        for (int i = enemies.size - 1; i >= 0; i--) {
            if (enemies.get(i).y + enemies.get(i).height < 0)
                enemies.removeIndex(i);
        }
        for (int i = hearts.size - 1; i >= 0; i--) {
            if (hearts.get(i).y + hearts.get(i).height < 0)
                hearts.removeIndex(i);
        }
    }

    @Override
    public void dispose () {
        batch.dispose();
        playerTexture.dispose();
        enemyTexture.dispose();
        heartTexture.dispose();
        bulletTexture.dispose();
    }
}
