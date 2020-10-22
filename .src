import arcade
import os

# Constants
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
SCREEN_TITLE = "2D Platform Game"

# Constants used to scale our sprites from their original size
CHARACTER_SCALING = 0.5
TILE_SCALING = 0.3
COIN_SCALING = 0.3
SPRITE_PIXEL_SIZE = 128
GRID_PIXEL_SIZE = (SPRITE_PIXEL_SIZE * TILE_SCALING)
SPRITE_SCALING = 0.3
SPRITE_SIZE = int(SPRITE_PIXEL_SIZE * SPRITE_SCALING)

# Speed of player
PLAYER_MOVEMENT_SPEED = 4
GRAVITY = 1
PLAYER_JUMP_SPEED = 17

# Pixel Scrolling

LEFT_VIEWPORT_MARGIN = 250
RIGHT_VIEWPORT_MARGIN = 250
BOTTOM_VIEWPORT_MARGIN = 100
TOP_VIEWPORT_MARGIN = 50


PLAYER_START_X = SPRITE_PIXEL_SIZE * TILE_SCALING * 1.7
PLAYER_START_Y = 400

# Facing constants
RIGHT_FACING = 0
LEFT_FACING = 1


def load_texture_pair(filename):
    return [
        arcade.load_texture(filename), arcade.load_texture(filename, mirrored=True)
    ]


class PlayerCharacter(arcade.Sprite):

    def __init__(self):

        # Set up parent class
        super().__init__()

        # Default to face-right
        self.character_face_direction = RIGHT_FACING

        # Used for flipping between image sequences
        self.cur_texture = 0
        self.scale = CHARACTER_SCALING

        # Track our state
        self.jumping = False
        self.climbing = False
        self.is_on_ladder = False

        # Textures
        main_path = ":resources:images/animated_characters/male_person/malePerson"

        # Textures for idle standing
        self.idle_texture_pair = load_texture_pair(f"{main_path}_idle.png")
        self.jump_texture_pair = load_texture_pair(f"{main_path}_jump.png")
        self.fall_texture_pair = load_texture_pair(f"{main_path}_fall.png")

        # Textures for walking
        self.walk_textures = []
        for i in range(8):
            texture = load_texture_pair(f"{main_path}_walk{i}.png")
            self.walk_textures.append(texture)

        # Textures for climbing
        self.climbing_textures = []
        texture = arcade.load_texture(f"{main_path}_climb0.png")
        self.climbing_textures.append(texture)
        texture = arcade.load_texture(f"{main_path}_climb1.png")
        self.climbing_textures.append(texture)

        # Initial texture (go to 0)
        self.texture = self.idle_texture_pair[0]
        # self.set_hit_box(self.texture.hit_box_points)

    def update_animation(self, delta_time: float = 1 / 60):

        # Face left / right
        if self.change_x < 0 and self.character_face_direction == RIGHT_FACING:
            self.character_face_direction = LEFT_FACING
        elif self.change_x > 0 and self.character_face_direction == LEFT_FACING:
            self.character_face_direction = RIGHT_FACING

        # Climbing animation
        if self.is_on_ladder:
            self.climbing = True
        if not self.is_on_ladder and self.climbing:
            self.climbing = False
        if self.climbing and abs(self.change_y) > 1:
            self.cur_texture += 1
            if self.cur_texture > 7:
                self.cur_texture = 0
        if self.climbing:
            self.texture = self.climbing_textures[self.cur_texture // 4]
            return

        # jump face
        if self.change_y > 0 and not self.is_on_ladder:
            self.texture = self.jump_texture_pair[self.character_face_direction]
            return
        elif self.change_y < 0 and not self.is_on_ladder:
            self.texture = self.fall_texture_pair[self.character_face_direction]
            return

        # idle animation
        if self.change_x == 0:
            self.texture = self.idle_texture_pair[self.character_face_direction]
            return

        # walking animation
        self.cur_texture += 1
        if self.cur_texture > 7:
            self.cur_texture = 0
        self.texture = self.walk_textures[self.cur_texture][self.character_face_direction]


class MenuView(arcade.View):
    def on_show(self):
        arcade.set_background_color(arcade.color.BLUE_GREEN)

    def on_draw(self):
        arcade.start_render()
        arcade.draw_text("Menu Screen - SPACE to start", SCREEN_WIDTH//2, SCREEN_HEIGHT//2,
                         arcade.color.BLACK, font_size=30, anchor_x="center")

    def on_key_press(self, key, modifiers):
        if key == arcade.key.SPACE:
            game_view = GameView()
            game_view.setup()
            self.window.show_view(game_view)


class GameView(arcade.View):
    """
    Main application class.
    """

    def __init__(self):

        # Call the parent class and set up the window
        super().__init__()

        # Setting the path to start with this program
        file_path = os.path.dirname(os.path.abspath(__file__))
        os.chdir(file_path)

        # Track the current state of what key is pressed
        self.left_pressed = False
        self.right_pressed = False
        self.up_pressed = False
        self.down_pressed = False
        self.jump_needs_reset = False

        self.coin_list = None
        self.wall_list = None
        self.background_list = None
        self.ladder_list = None
        self.player_list = None
        self.dont_touch_list = None
        self.foreground_list = None
        self.enemy_list = None

        # Separate variable that holds the player sprite
        self.player_sprite = None

        # Our engine
        self.physics_engine = None

        self.view_bottom = 0
        self.view_left = 0

        self.end_of_map = 0
        self.background = None

        # Level
        self.level = 1

        # Our score
        self.score = 0
        self.game_score = 0

        # Player Health
        self.health = 3

        # time
        self.total_time = 301.0

        self.game_over = False

        # Load sounds
        self.collect_coin_sound = arcade.load_sound(":resources:sounds/coin1.wav")
        self.jump_sound = arcade.load_sound(":resources:sounds/jump1.wav")
        self.game_sound = arcade.load_sound(":resources:sounds/Child's Nightmare.ogg")
        self.game_over = arcade.load_sound(":resources:sounds/gameover1.wav")
        self.level_update_sound = arcade.load_sound(":resources:sounds/upgrade5.wav")

    def setup(self, level=1):
        """ Set up the game here. Call this function to restart the game. """

        # arcade.play_sound(self.game_sound)

        self.view_bottom = 0
        self.view_left = 0

        self.score += self.game_score

        # Create the Sprite lists
        self.player_list = arcade.SpriteList()
        self.background_list = arcade.SpriteList()
        self.wall_list = arcade.SpriteList()
        self.coin_list = arcade.SpriteList()
        self.enemy_list = arcade.SpriteList()

        # Set up the player
        self.player_sprite = PlayerCharacter()
        self.player_sprite.center_x = PLAYER_START_X
        self.player_sprite.center_y = PLAYER_START_Y
        self.player_list.append(self.player_sprite)

        # We will add the map here...

        platforms_layer_name = 'Platforms'
        coins_layer_name = 'Coins'
        moving_platforms_layer_name = 'Moving Platforms'
        # forenground_layer_name = 'Foreground'
        dont_touch_layer_name = "Don't Touch"

        # enemy1
        enemy = arcade.Sprite(":resources:images/enemies/wormGreen.png", SPRITE_SCALING)
        # Draw enemy1
        enemy.bottom = SPRITE_SIZE * 3.2
        enemy.left = SPRITE_SIZE * 2.5

        enemy.boundary_right = SPRITE_SIZE * 16
        enemy.boundary_left = SPRITE_SIZE * 2.5
        enemy.change_x = 5
        self.enemy_list.append(enemy)

        # enemy2
        enemy = arcade.Sprite(":resources:images/enemies/slimeGreen.png", SPRITE_SCALING)
        # draw enemy2
        enemy.bottom = SPRITE_SIZE * 10.2
        enemy.left = SPRITE_SIZE * 61.5

        enemy.boundary_right = SPRITE_SIZE * 73
        enemy.boundary_left = SPRITE_SIZE * 61.5
        enemy.change_x = 8
        self.enemy_list.append(enemy)

        # enemy 3
        enemy = arcade.Sprite(":resources:images/enemies/bee.png", SPRITE_SCALING)
        # draw enemy 3
        enemy.bottom = SPRITE_SIZE * 3.7
        enemy.left = SPRITE_SIZE * 24

        enemy.boundary_right = SPRITE_SIZE * 39
        enemy.boundary_left = SPRITE_SIZE * 24
        enemy.change_x = 7
        self.enemy_list.append(enemy)

        # Map and tools
        map_name = f":resources:tmx_maps/Admap_level_{level}.tmx"
        my_map = arcade.tilemap.read_tmx(map_name)
        self.end_of_map = my_map.map_size.width * GRID_PIXEL_SIZE
        # PLATFORMS
        self.wall_list = arcade.tilemap.process_layer(my_map, platforms_layer_name, TILE_SCALING)

        # Coins
        self.coin_list = arcade.tilemap.process_layer(my_map, coins_layer_name, TILE_SCALING)

        # Moving platforms(if we have)
        moving_platforms_list = arcade.tilemap.process_layer(my_map, moving_platforms_layer_name, TILE_SCALING)
        for sprite in moving_platforms_list:
            self.wall_list.append(sprite)

        # Background Objects
        # self.background_list = arcade.tilemap.process_layer(my_map, "Background", TILE_SCALING)
        self.ladder_list = arcade.tilemap.process_layer(my_map, "Ladders", TILE_SCALING)
        # self.background = arcade.load_texture(":resources:images/backgrounds/back1.jpg")

        # Don't Touch
        self.dont_touch_list = arcade.tilemap.process_layer(my_map, dont_touch_layer_name, TILE_SCALING)

        # For other stuffs
        arcade.set_background_color(arcade.color.BLACK)
        # Physics Engine
        self.physics_engine = arcade.PhysicsEnginePlatformer(self.player_sprite, self.wall_list,
                                                             gravity_constant=GRAVITY, ladders=self.ladder_list)

    def on_draw(self):
        """ Render the screen. """

        # Clear the screen to the background color
        arcade.start_render()

        # scale = SCREEN_WIDTH / self.background.width
        # arcade.draw_lrwh_rectangle_textured(-500, -300, 12800, 2560, self.background)
        # Draw our sprites
        self.wall_list.draw()
        self.background_list.draw()
        self.ladder_list.draw()
        self.coin_list.draw()
        self.player_list.draw()
        self.dont_touch_list.draw()
        self.enemy_list.draw()

        # Calculating time
        minutes = int(self.total_time) // 60
        seconds = int(self.total_time) % 60
        score_time = f"    {minutes:02d}:{seconds:02d}"

        # for Showing time
        arcade.draw_text(score_time, 350 + self.view_left, 570 + self.view_bottom, arcade.csscolor.WHITE, 18)

        # For Showing Score
        score_text = f"Score: {self.game_score}"
        arcade.draw_text(score_text, 10 + self.view_left, 10 + self.view_bottom, arcade.csscolor.WHITE, 18)

        # For showing Health
        score_health = f"Health: {self.health}"
        arcade.draw_text(score_health, 10 + self.view_left, 570 + self.view_bottom, arcade.csscolor.WHITE, 18)

        # Draw hit boxes. (after create map)
        # for wall in self.wall_list:
        #     wall.draw_hit_box(arcade.color.BLACK, 3)
        #
        # self.player_sprite.draw_hit_box(arcade.color.RED, 3)

    def process_keychange(self):
        # Called when we change a key up/down or we move on/off a ladder.

        # process up/down
        if self.up_pressed and not self.down_pressed:
            if self.physics_engine.is_on_ladder():
                self.player_sprite.change_y = PLAYER_MOVEMENT_SPEED
            elif self.physics_engine.can_jump() and not self.jump_needs_reset:
                self.player_sprite.change_y = PLAYER_JUMP_SPEED
                self.jump_needs_reset = True
                arcade.play_sound(self.jump_sound)
        elif self.down_pressed and not self.up_pressed:
            if self.physics_engine.is_on_ladder():
                self.player_sprite.change_y = -PLAYER_MOVEMENT_SPEED

        # Process up/down when no movement
        if self.physics_engine.is_on_ladder():
            if not self.up_pressed and not self.down_pressed:
                self.player_sprite.change_y = 0
            elif self.up_pressed and self.down_pressed:
                self.player_sprite.change_y = 0

        # process left/right
        if self.right_pressed and not self.left_pressed:
            self.player_sprite.change_x = PLAYER_MOVEMENT_SPEED
        elif self.left_pressed and not self.right_pressed:
            self.player_sprite.change_x = -PLAYER_MOVEMENT_SPEED
        else:
            self.player_sprite.change_x = 0

    def on_key_press(self, key, modifiers):  # Keyboard functions

        if key == arcade.key.UP or key == arcade.key.W:
            self.up_pressed = True
        elif key == arcade.key.DOWN or key == arcade.key.S:
            self.down_pressed = True
        elif key == arcade.key.LEFT or key == arcade.key.A:
            self.left_pressed = True
        elif key == arcade.key.RIGHT or key == arcade.key.D:
            self.right_pressed = True

        self.process_keychange()

    def on_key_release(self, key, modifiers):

        if key == arcade.key.UP or key == arcade.key.W:
            self.up_pressed = False
            self.jump_needs_reset = False
        elif key == arcade.key.DOWN or key == arcade.key.S:
            self.down_pressed = False
        elif key == arcade.key.LEFT or key == arcade.key.A:
            self.left_pressed = False
        elif key == arcade.key.RIGHT or key == arcade.key.D:
            self.right_pressed = False

        self.process_keychange()

    def on_update(self, delta_time):
        # We're calling physics engine
        # Enemy
        if not self.health == 0:
            self.enemy_list.update()

        self.physics_engine.update()

        self.total_time -= delta_time

        # Update animations
        if self.physics_engine.can_jump():
            self.player_sprite.can_jump = False
        else:
            self.player_sprite.can_jump = True

        if self.physics_engine.is_on_ladder() and not self.physics_engine.can_jump():
            self.player_sprite.is_on_ladder = True
            self.process_keychange()
        else:
            self.player_sprite.is_on_ladder = False
            self.process_keychange()

        self.coin_list.update_animation(delta_time)
        self.background_list.update_animation(delta_time)
        self.player_list.update_animation(delta_time)

        self.wall_list.update()

        # See if the moving wall hit a boundary and needs to reverse direction.
        for wall in self.wall_list:

            if wall.boundary_right and wall.right > wall.boundary_right and wall.change_x > 0:
                wall.change_x *= -1
            if wall.boundary_left and wall.left < wall.boundary_left and wall.change_x < 0:
                wall.change_x *= -1
            if wall.boundary_top and wall.top > wall.boundary_top and wall.change_y > 0:
                wall.change_y *= -1
            if wall.boundary_bottom and wall.bottom < wall.boundary_bottom and wall.change_y < 0:
                wall.change_y *= -1

        # if you hit any coins
        coin_hit_list = arcade.check_for_collision_with_list(self.player_sprite, self.coin_list)

        for coin in coin_hit_list:
            self.game_score += 1
            # Remove the coin
            coin.remove_from_sprite_lists()
            # Play sound
            arcade.play_sound(self.collect_coin_sound)

        # enemy
        for enemy in self.enemy_list:
            if len(arcade.check_for_collision_with_list(enemy, self.wall_list)) > 0:
                enemy.change_x *= -1
            elif enemy.boundary_left is not None and enemy.left < enemy.boundary_left:
                enemy.change_x *= -1
            elif enemy.boundary_right is not None and enemy.right > enemy.boundary_right:
                enemy.change_x *= -1

        changed_viewport = False

        if self.player_sprite.center_x >= self.end_of_map:
            self.level += 1
            self.setup(self.level)
            self.view_left = 0
            self.view_bottom = 0
            arcade.stop_sound(self.game_sound)
            arcade.play_sound(self.level_update_sound)
            # arcade.play_sound(self.game_sound)
            changed_viewport = True

        # Manage Scrolling

        if len(arcade.check_for_collision_with_list(self.player_sprite, self.enemy_list)) > 0:
            self.player_sprite.center_x = PLAYER_START_X
            self.player_sprite.center_y = PLAYER_START_Y

            self.view_left = 0
            self.view_bottom = 0
            changed_viewport = True
            arcade.play_sound(self.game_over)
            self.health -= 1

        # if player falls
        if self.player_sprite.center_y < -100:
            self.player_sprite.center_x = PLAYER_START_X
            self.player_sprite.center_y = PLAYER_START_Y

            # Set the camera
            self.view_left = 0
            self.view_bottom = 0
            changed_viewport = True
            arcade.play_sound(self.game_over)
            self.health -= 1

        if self.health == 0:
            arcade.stop_sound(self.game_sound)
            arcade.pause(3)
            arcade.play_sound(self.game_sound)
            self.total_time = 303.0
            self.view_left = 0
            self.view_bottom = 0
            changed_viewport = True
            self.health = 3
            self.game_score = self.game_score - 20

        # end of the game
        if self.level > 3:
            game_over_view = GameOverView()
            self.window.show_view(game_over_view)

        # if character hits don't touch
        if arcade.check_for_collision_with_list(self.player_sprite, self.dont_touch_list):
            self.player_sprite.change_x = 0
            self.player_sprite.change_y = 0
            self.player_sprite.center_x = PLAYER_START_X
            self.player_sprite.center_y = PLAYER_START_Y

            # Set the camera to the start
            self.view_left = 0
            self.view_bottom = 0
            changed_viewport = True
            arcade.play_sound(self.game_over)
            self.health -= 1

        # Scroll left
        left_boundary = self.view_left + LEFT_VIEWPORT_MARGIN
        if self.player_sprite.left < left_boundary:
            self.view_left -= left_boundary - self.player_sprite.left
            changed_viewport = True

        # Scroll right
        right_boundary = self.view_left + SCREEN_WIDTH - RIGHT_VIEWPORT_MARGIN
        if self.player_sprite.right > right_boundary:
            self.view_left += self.player_sprite.right - right_boundary
            changed_viewport = True

        # Scroll up
        top_boundary = self.view_bottom + SCREEN_HEIGHT - TOP_VIEWPORT_MARGIN
        if self.player_sprite.top > top_boundary:
            self.view_bottom += self.player_sprite.top - top_boundary
            changed_viewport = True

        # Scroll down
        bottom_boundary = self.view_bottom + BOTTOM_VIEWPORT_MARGIN
        if self.player_sprite.bottom < bottom_boundary:
            self.view_bottom -= bottom_boundary - self.player_sprite.bottom
            changed_viewport = True

        if changed_viewport:
            self.view_bottom = int(self.view_bottom)
            self.view_left = int(self.view_left)

            # Done the Scrolling

            arcade.set_viewport(self.view_left, SCREEN_WIDTH + self.view_left, self.view_bottom,
                                SCREEN_HEIGHT + self.view_bottom)


class GameOverView(arcade.View):

    def on_show(self):
        arcade.set_background_color(arcade.color.BLACK)

    def on_draw(self):
        arcade.start_render()

        arcade.draw_text("Game Over - press ESCAPE to advance", SCREEN_WIDTH//4.2, SCREEN_HEIGHT//2,
                         arcade.color.WHITE, font_size=30)

    def on_key_press(self, key, _modifiers):
        if key == arcade.key.ESCAPE:
            menu_view = MenuView()
            self.window.show_view(menu_view)


def main():
    """ Main method """
    window = arcade.Window(SCREEN_WIDTH, SCREEN_HEIGHT)
    menu_view = MenuView()
    window.show_view(menu_view)
    arcade.run()


if __name__ == "__main__":
    main()
