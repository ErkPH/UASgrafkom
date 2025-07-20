from OpenGL.GL import *
from OpenGL.GLU import *
from OpenGL.GLUT import *
import sys, math

# Konstanta & Global State
SCREEN_W, SCREEN_H = 800, 600
current_mode = "draw_point"
draw_items = []
temp_buffer = []
clip_rect = []
# --- PERUBAHAN 1: Warna default objek 2D menjadi putih ---
default_color = (1.0, 1.0, 1.0) # Warna default objek 2D menjadi putih
user_color = default_color
thickness = 2.0
angle_3d = 0.0 
angle_2d = 0.0
camera_z = 10.0 
transform_flag = None
translate_offset = [0.0, 0.0]
selected_index = -1
show_3d_cube = False

# Variabel Global untuk Kubus 3D (Translasi dan Rotasi Manual)
cube_translate_x = 0.0
cube_translate_y = 0.0
cube_translate_z = 0.0
cube_angle_x = 0.0
cube_angle_y = 0.0
cube_angle_z = 0.0

# Variabel Global untuk Rotasi Otomatis Kubus (Spin)
auto_spin_cube = False
spin_speed = 0.5 


# ===================== UTILITAS CLIP =====================
def is_inside_clip(x, y):
    if len(clip_rect) == 2:
        x1, y1 = clip_rect[0]
        x2, y2 = clip_rect[1]
        return min(x1, x2) <= x <= max(x1, x2) and min(y1, y2) <= y <= max(y1, y2)
    return True

def render_clip_border():
    if len(clip_rect) == 2:
        x1, y1 = clip_rect[0]
        x2, y2 = clip_rect[1]
        glColor3f(0.5, 0.5, 0.5)
        glBegin(GL_LINE_LOOP)
        glVertex2f(x1, y1)
        glVertex2f(x2, y1)
        glVertex2f(x2, y2)
        glVertex2f(x1, y2)
        glEnd()

# ===================== ALGORITMA CLIPPING GARIS =====================
def cohen_sutherland_clip(x1, y1, x2, y2):
    INSIDE, LEFT, RIGHT, BOTTOM, TOP = 0, 1, 2, 4, 8

    def compute_outcode(x, y):
        x_min, y_min = min(clip_rect[0][0], clip_rect[1][0]), min(clip_rect[0][1], clip_rect[1][1])
        x_max, y_max = max(clip_rect[0][0], clip_rect[1][0]), max(clip_rect[0][1], clip_rect[1][1])
        code = INSIDE
        if x < x_min: code |= LEFT
        elif x > x_max: code |= RIGHT
        if y < y_min: code |= BOTTOM
        elif y > y_max: code |= TOP
        return code

    outcode1 = compute_outcode(x1, y1)
    outcode2 = compute_outcode(x2, y2)

    while True:
        if not (outcode1 | outcode2):
            return (x1, y1, x2, y2)
        elif outcode1 & outcode2:
            return None
        else:
            outcode_out = outcode1 if outcode1 else outcode2
            x_min, y_min = min(clip_rect[0][0], clip_rect[1][0]), min(clip_rect[0][1], clip_rect[1][1])
            x_max, y_max = max(clip_rect[0][0], clip_rect[1][0]), max(clip_rect[0][1], clip_rect[1][1])

            if outcode_out & TOP:
                x = x1 + (x2 - x1) * (y_max - y1) / (y2 - y1)
                y = y_max
            elif outcode_out & BOTTOM:
                x = x1 + (x2 - x1) * (y_min - y1) / (y2 - y1)
                y = y_min
            elif outcode_out & RIGHT:
                y = y1 + (y2 - y1) * (x_max - x1) / (x2 - x1)
                x = x_max
            elif outcode_out & LEFT:
                y = y1 + (y2 - y1) * (x_min - x1) / (x2 - x1)
                x = x_min

            if outcode_out == outcode1:
                x1, y1 = x, y
                outcode1 = compute_outcode(x1, y1)
            else:
                x2, y2 = x, y
                outcode2 = compute_outcode(x2, y2)

# ===================== RENDERING OBJEK 2D =====================
def render_2d_objects():
    glDisable(GL_LIGHTING) 
    for idx, item in enumerate(draw_items):
        shape, data = item["type"], item["data"]
        glLineWidth(item.get("line", 1.0))
        color = item["color"]

        glPushMatrix() 
        glColor3f(*color)

        if idx == selected_index and transform_flag and shape != "point":
            if transform_flag == "translate":
                glTranslatef(translate_offset[0], translate_offset[1], 0)
            elif transform_flag == "scale":
                if shape == "square" or shape == "ellipse":
                    obj_center_x, obj_center_y = data[0], data[1]
                elif shape == "line":
                    obj_center_x = (data[0] + data[2]) / 2
                    obj_center_y = (data[1] + data[3]) / 2
                else: 
                    obj_center_x, obj_center_y = data[0], data[1]

                glTranslatef(obj_center_x, obj_center_y, 0)
                glScalef(1.3, 1.3, 1)
                glTranslatef(-obj_center_x, -obj_center_y, 0)

            elif transform_flag == "rotate":
                if shape == "square" or shape == "ellipse":
                    obj_center_x, obj_center_y = data[0], data[1]
                elif shape == "line":
                    obj_center_x = (data[0] + data[2]) / 2
                    obj_center_y = (data[1] + data[3]) / 2
                else: 
                    obj_center_x, obj_center_y = data[0], data[1]

                glTranslatef(obj_center_x, obj_center_y, 0)
                glRotatef(angle_2d, 0, 0, 1)
                glTranslatef(-obj_center_x, -obj_center_y, 0)

        if shape == "point":
            if not is_inside_clip(*data):
                glPopMatrix()
                continue
            glPointSize(item.get("line", 5.0))
            glBegin(GL_POINTS)
            glVertex2f(*data)
            glEnd()

        elif shape == "line" and len(data) == 4:
            if len(clip_rect) == 2:
                clipped = cohen_sutherland_clip(*data)
                if clipped:
                    glColor3f(0.0, 1.0, 0.0) 
                    glBegin(GL_LINES)
                    glVertex2f(clipped[0], clipped[1])
                    glVertex2f(clipped[2], clipped[3])
                    glEnd()
            else:
                glBegin(GL_LINES)
                glVertex2f(data[0], data[1])
                glVertex2f(data[2], data[3])
                glEnd()

        elif shape == "square":
            x, y = data
            s = 50
            glBegin(GL_LINE_LOOP)
            glVertex2f(x, y)
            glVertex2f(x + s, y)
            glVertex2f(x + s, y + s)
            glVertex2f(x, y + s)
            glEnd()

        elif shape == "ellipse":
            x, y = data
            glBegin(GL_LINE_LOOP)
            for i in range(100):
                t = 2 * math.pi * i / 100
                glVertex2f(x + 30 * math.cos(t), y + 20 * math.sin(t))
            glEnd()
        glPopMatrix()

    render_clip_border()

# ===================== RENDERING 3D =====================
def render_3d_cube():
    glPushMatrix()
    glTranslatef(cube_translate_x, cube_translate_y, -3.0 + cube_translate_z)
    glRotatef(cube_angle_x, 1, 0, 0)
    glRotatef(cube_angle_y, 0, 1, 0)
    glRotatef(cube_angle_z, 0, 0, 1)

    glutSolidCube(1.0) # Gambar kubus
    glPopMatrix()

def apply_lighting():
    glEnable(GL_LIGHTING)
    glEnable(GL_LIGHT0)

    # Properti Sumber Cahaya (Light0)
    light_ambient = [0.2, 0.2, 0.2, 1.0] # Cahaya ambient cukup terang
    light_diffuse = [0.7, 0.7, 0.7, 1.0]
    light_specular = [1.0, 1.0, 1.0, 1.0]
    light_position = [2.0, 2.0, 2.0, 1.0]

    glLightfv(GL_LIGHT0, GL_AMBIENT, light_ambient)
    glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse)
    glLightfv(GL_LIGHT0, GL_SPECULAR, light_specular)
    glLightfv(GL_LIGHT0, GL_POSITION, light_position)

    # Properti Material Kubus (Model Pencahayaan Phong)
    material_ambient = [0.8, 0.8, 0.8, 1.0] # --- PERUBAHAN 3: Ambient material kubus lebih terang agar terlihat jelas di background gelap ---
    material_diffuse = [1.0, 1.0, 1.0, 1.0] # Warna dasar kubus putih
    material_specular = [0.5, 0.5, 0.5, 1.0]
    material_shininess = [50.0]

    glMaterialfv(GL_FRONT_AND_BACK, GL_AMBIENT, material_ambient)
    glMaterialfv(GL_FRONT_AND_BACK, GL_DIFFUSE, material_diffuse)
    glMaterialfv(GL_FRONT_AND_BACK, GL_SPECULAR, material_specular)
    glMaterialfv(GL_FRONT_AND_BACK, GL_SHININESS, material_shininess)

    glEnable(GL_NORMALIZE)
    glDisable(GL_COLOR_MATERIAL)


# ===================== FUNGSI IDLE UNTUK ANIMASI =====================
def idle():
    global cube_angle_y 
    if show_3d_cube:
        if auto_spin_cube:
            cube_angle_y += spin_speed
            if cube_angle_y > 360: cube_angle_y -= 360 
    glutPostRedisplay() 

# ===================== OPENGL DISPLAY DAN MAIN LOOP =====================
def display():
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

    # --- Mode 2D ---
    glMatrixMode(GL_PROJECTION)
    glLoadIdentity()
    gluOrtho2D(0, SCREEN_W, 0, SCREEN_H)
    glMatrixMode(GL_MODELVIEW)
    glLoadIdentity()

    if len(clip_rect) == 2:
        x1, y1 = clip_rect[0]
        x2, y2 = clip_rect[1]
        sx = int(min(x1, x2))
        sy = int(min(y1, y2))
        sw = int(abs(x2 - x1))
        sh = int(abs(y2 - y1))
        glEnable(GL_SCISSOR_TEST)
        glScissor(sx, sy, sw, sh)

    render_2d_objects()

    glDisable(GL_SCISSOR_TEST)

    # --- Mode 3D ---
    if show_3d_cube:
        glMatrixMode(GL_PROJECTION)
        glLoadIdentity()
        gluPerspective(45, SCREEN_W / SCREEN_H, 1, 100)
        glMatrixMode(GL_MODELVIEW)
        glLoadIdentity()
        gluLookAt(2, 0, camera_z, 0, 0, -3, 0, 1, 0)

        glEnable(GL_DEPTH_TEST)
        apply_lighting()
        render_3d_cube()
    else:
        glDisable(GL_DEPTH_TEST)
        glDisable(GL_LIGHTING)

    glutSwapBuffers()

def mouse_click(button, state, x, y):
    global temp_buffer, clip_rect, selected_index
    if state == GLUT_DOWN:
        x, y = x, SCREEN_H - y
        if current_mode == "draw_point":
            draw_items.append({"type": "point", "data": (x, y), "color": user_color, "line": thickness})
            selected_index = len(draw_items) - 1
        elif current_mode == "draw_line":
            temp_buffer.append((x, y))
            if len(temp_buffer) == 2:
                x1, y1 = temp_buffer[0]
                x2, y2 = temp_buffer[1]
                draw_items.append({"type": "line", "data": (x1, y1, x2, y2), "color": user_color, "line": thickness})
                selected_index = len(draw_items) - 1
                temp_buffer = []
        elif current_mode == "draw_square":
            draw_items.append({"type": "square", "data": (x, y), "color": user_color, "line": thickness})
            selected_index = len(draw_items) - 1
        elif current_mode == "draw_ellipse":
            draw_items.append({"type": "ellipse", "data": (x, y), "color": user_color, "line": thickness})
            selected_index = len(draw_items) - 1
        elif current_mode == "clip_area":
            clip_rect.append((x, y))
            if len(clip_rect) > 2:
                clip_rect = clip_rect[-2:]
        glutPostRedisplay()

def special_keys(key, x, y):
    global camera_z
    if key == GLUT_KEY_UP:
        camera_z -= 0.2
    elif key == GLUT_KEY_DOWN:
        camera_z += 0.2
    glutPostRedisplay()

def init_display():
    # --- PERUBAHAN 2: Warna latar belakang menjadi hitam ---
    glClearColor(0.0, 0.0, 0.0, 1.0) # Warna latar belakang hitam

def main():
    glutInit(sys.argv)
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH)
    glutInitWindowSize(SCREEN_W, SCREEN_H)
    glutCreateWindow(b"Studio Grafika 2D/3D - Final")
    glutDisplayFunc(display)
    glutMouseFunc(mouse_click)
    glutKeyboardFunc(keyboard_input)
    glutSpecialFunc(special_keys)
    glutIdleFunc(idle)
    init_display()
    glutMainLoop()

def keyboard_input(key, x, y):
    global current_mode, user_color, thickness, angle_3d, angle_2d, camera_z, transform_flag, translate_offset, selected_index, show_3d_cube
    global cube_translate_x, cube_translate_y, cube_translate_z, cube_angle_x, cube_angle_y, cube_angle_z
    global auto_spin_cube, spin_speed 

    # Mode Gambar 2D
    if key == b'1':
        current_mode = "draw_point"
        selected_index = -1
    elif key == b'2':
        current_mode = "draw_line"
        selected_index = -1
    elif key == b'3':
        current_mode = "draw_square"
        selected_index = -1
    elif key == b'4':
        current_mode = "draw_ellipse"
        selected_index = -1
    elif key == b'5':
        current_mode = "clip_area"
        selected_index = -1
        clip_rect.clear() 

    # Pengaturan Warna
    elif key == b'r': user_color = (0.9, 0.1, 0.1)
    elif key == b'g': user_color = (0.1, 0.8, 0.1)
    elif key == b'b': user_color = (0.2, 0.2, 1.0)
    elif key == b'd': user_color = default_color

    # Ketebalan Garis/Ukuran Titik
    elif key == b'+': thickness += 1.0
    elif key == b'-': thickness = max(1.0, thickness - 1.0)

    # Transformasi Objek 2D
    elif key == b't': 
        transform_flag = "translate"
    elif key == b'o': 
        transform_flag = "rotate"
    elif key == b'S': 
        transform_flag = "scale"
    elif key == b'c': 
        transform_flag = None
        translate_offset = [0.0, 0.0]
        angle_2d = 0.0
    elif key == b'q': 
        angle_2d += 5

    # Pemilihan Objek 2D
    elif key == b'k': 
        if len(draw_items) > 0:
            selected_index = (selected_index - 1) % len(draw_items)
    elif key == b'l': 
        if len(draw_items) > 0:
            selected_index = (selected_index + 1) % len(draw_items)
    
    # Translasi Objek 2D terpilih (saat mode 't' aktif)
    elif key == b'y': 
        if selected_index != -1 and transform_flag == "translate":
            item = draw_items[selected_index]
            if item["type"] == "point":
                item["data"] = (item["data"][0], item["data"][1] + 5)
            elif item["type"] == "line":
                item["data"] = (item["data"][0], item["data"][1] + 5, item["data"][2], item["data"][3] + 5)
            elif item["type"] in ["square", "ellipse"]:
                item["data"] = (item["data"][0], item["data"][1] + 5)
    elif key == b'h': 
        if selected_index != -1 and transform_flag == "translate":
            item = draw_items[selected_index]
            if item["type"] == "point":
                item["data"] = (item["data"][0] - 5, item["data"][1])
            elif item["type"] == "line":
                item["data"] = (item["data"][0] - 5, item["data"][1], item["data"][2] - 5, item["data"][3])
            elif item["type"] in ["square", "ellipse"]:
                item["data"] = (item["data"][0] - 5, item["data"][1])
    elif key == b'j': 
        if selected_index != -1 and transform_flag == "translate":
            item = draw_items[selected_index]
            if item["type"] == "point":
                item["data"] = (item["data"][0], item["data"][1] - 5)
            elif item["type"] == "line":
                item["data"] = (item["data"][0], item["data"][1] - 5, item["data"][2], item["data"][3] - 5)
            elif item["type"] in ["square", "ellipse"]:
                item["data"] = (item["data"][0], item["data"][1] - 5)
    elif key == b'u': 
        if selected_index != -1 and transform_flag == "translate":
            item = draw_items[selected_index]
            if item["type"] == "point":
                item["data"] = (item["data"][0] + 5, item["data"][1])
            elif item["type"] == "line":
                item["data"] = (item["data"][0] + 5, item["data"][1], item["data"][2] + 5, item["data"][3])
            elif item["type"] in ["square", "ellipse"]:
                item["data"] = (item["data"][0] + 5, item["data"][1])

    # Kontrol Kubus 3D
    elif key == b'v': 
        show_3d_cube = not show_3d_cube
    elif key == b'[': 
        cube_translate_x -= 0.1
    elif key == b']': 
        cube_translate_x += 0.1
    elif key == b'p': 
        cube_translate_y += 0.1
    elif key == b';': 
        cube_translate_y -= 0.1
    elif key == b'.': 
        cube_translate_z += 0.1
    elif key == b'/': 
        cube_translate_z -= 0.1
    
    # Rotasi Manual Kubus
    elif key == b'f': 
        cube_angle_x += 5.0
    elif key == b'g': 
        cube_angle_y += 5.0
    elif key == b'H': 
        cube_angle_z += 5.0
    
    # Rotasi Otomatis Kubus (Spin)
    elif key == b'A': 
        auto_spin_cube = not auto_spin_cube
    
    elif key == b'0': 
        cube_translate_x = 0.0
        cube_translate_y = 0.0
        cube_translate_z = 0.0
        cube_angle_x = 0.0
        cube_angle_y = 0.0
        cube_angle_z = 0.0
        auto_spin_cube = False 

    glutPostRedisplay()

# ===================== PEMANGGILAN MAIN =====================
if __name__ == "__main__":
    main()
