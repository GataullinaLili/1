# 1
1
Вот полное решение лабораторной работы №5. Все скрипты написаны для Blender и используют его API.

1. Скрипт для генерации 10 случайных мешей и их анимации

```python
import bpy
import random
import math

def clear_scene():
    """Очищает сцену от всех объектов, кроме камеры и источника света"""
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete(use_global=False)

def generate_random_meshes(count=10):
    """Генерирует случайные меши"""
    meshes = []
    mesh_types = ['primitive_cube_add', 'primitive_cylinder_add', 'primitive_sphere_add', 
                  'primitive_cone_add', 'primitive_torus_add']
    
    for i in range(count):
        # Случайные координаты от -50 до 50
        x = random.uniform(-50, 50)
        y = random.uniform(-50, 50)
        z = random.uniform(-50, 50)
        
        # Случайный размер от 0.5 до 3
        size = random.uniform(0.5, 3)
        
        # Случайный тип меша
        mesh_type = random.choice(mesh_types)
        
        # Создание меша в зависимости от типа
        if mesh_type == 'primitive_cube_add':
            bpy.ops.mesh.primitive_cube_add(size=size, location=(x, y, z))
        elif mesh_type == 'primitive_cylinder_add':
            bpy.ops.mesh.primitive_cylinder_add(radius=size/2, depth=size, location=(x, y, z))
        elif mesh_type == 'primitive_sphere_add':
            bpy.ops.mesh.primitive_uv_sphere_add(radius=size/2, location=(x, y, z))
        elif mesh_type == 'primitive_cone_add':
            bpy.ops.mesh.primitive_cone_add(radius1=size/2, depth=size, location=(x, y, z))
        elif mesh_type == 'primitive_torus_add':
            bpy.ops.mesh.primitive_torus_add(major_radius=size/2, minor_radius=size/4, location=(x, y, z))
        
        obj = bpy.context.active_object
        obj.name = f"Random_Mesh_{i+1}"
        meshes.append(obj)
        
        # Добавление случайного материала
        mat = bpy.data.materials.new(name=f"Material_{obj.name}")
        mat.use_nodes = True
        mat.node_tree.nodes["Principled BSDF"].inputs[0].default_value = (
            random.random(), random.random(), random.random(), 1.0
        )
        obj.data.materials.append(mat)
    
    return meshes

def animate_to_target(meshes, target_location=(100, 100, 0), frame_duration=30):
    """Создает анимацию перемещения, поворота и масштабирования к целевой точке"""
    
    for i, obj in enumerate(meshes):
        # Сохраняем начальные значения
        start_location = obj.location.copy()
        start_rotation = obj.rotation_euler.copy()
        start_scale = obj.scale.copy()
        
        # Целевые значения
        target_scale = (0.5, 0.5, 0.5)  # Уменьшаем масштаб
        
        # Расчет угла поворота (поворот к целевой точке)
        direction = (target_location[0] - start_location[0], 
                    target_location[1] - start_location[1], 0)
        target_angle = math.atan2(direction[1], direction[0])
        target_rotation = (0, 0, target_angle)
        
        # Устанавливаем ключевые кадры для начала
        start_frame = i * frame_duration
        end_frame = start_frame + frame_duration
        
        # Начальные ключевые кадры
        obj.location = start_location
        obj.keyframe_insert(data_path="location", frame=start_frame)
        obj.rotation_euler = start_rotation
        obj.keyframe_insert(data_path="rotation_euler", frame=start_frame)
        obj.scale = start_scale
        obj.keyframe_insert(data_path="scale", frame=start_frame)
        
        # Конечные ключевые кадры
        obj.location = target_location
        obj.keyframe_insert(data_path="location", frame=end_frame)
        obj.rotation_euler = target_rotation
        obj.keyframe_insert(data_path="rotation_euler", frame=end_frame)
        obj.scale = target_scale
        obj.keyframe_insert(data_path="scale", frame=end_frame)

# Основная часть
clear_scene()
meshes = generate_random_meshes(10)
animate_to_target(meshes)
print(f"Сгенерировано {len(meshes)} мешей и создана анимация")
```

2. Скрипт для сохранения модели в текстовый файл

```python
import bpy
import json
import os

def export_model_to_text(filepath):
    """Экспортирует активную модель в текстовый файл"""
    
    if not bpy.context.active_object:
        print("Нет активного объекта для экспорта")
        return False
    
    obj = bpy.context.active_object
    
    # Собираем данные о модели
    model_data = {
        "name": obj.name,
        "type": obj.type,
        "location": list(obj.location),
        "rotation": list(obj.rotation_euler),
        "scale": list(obj.scale),
        "vertices": [],
        "faces": []
    }
    
    # Если это меш, собираем данные о вершинах и гранях
    if obj.type == 'MESH':
        mesh = obj.data
        
        # Сохраняем вершины
        for vertex in mesh.vertices:
            model_data["vertices"].append(list(vertex.co))
        
        # Сохраняем грани (полигоны)
        for polygon in mesh.polygons:
            face_vertices = [mesh.vertices[v].co[:] for v in polygon.vertices]
            model_data["faces"].append(face_vertices)
    
    # Сохраняем в JSON файл
    with open(filepath, 'w', encoding='utf-8') as f:
        json.dump(model_data, f, indent=4, ensure_ascii=False)
    
    print(f"Модель '{obj.name}' сохранена в файл: {filepath}")
    return True

# Пример использования
filepath = os.path.join(bpy.path.abspath("//"), "exported_model.txt")
export_model_to_text(filepath)
```

3. Скрипт для загрузки модели из текстового файла

```python
import bpy
import json
import os

def import_model_from_text(filepath):
    """Импортирует модель из текстового файла"""
    
    # Загрузка данных из файла
    with open(filepath, 'r', encoding='utf-8') as f:
        model_data = json.load(f)
    
    # Создание нового меша
    mesh = bpy.data.meshes.new(model_data["name"])
    obj = bpy.data.objects.new(model_data["name"], mesh)
    
    # Привязка объекта к сцене
    bpy.context.collection.objects.link(obj)
    
    # Создание геометрии
    if model_data["vertices"]:
        # Создание вершин и граней
        vertices = [tuple(v) for v in model_data["vertices"]]
        
        if model_data["faces"]:
            # Создание граней из данных
            faces = []
            for face in model_data["faces"]:
                # Находим индексы вершин для каждой грани
                face_indices = []
                for vertex in face:
                    for i, v in enumerate(vertices):
                        if (abs(v[0] - vertex[0]) < 0.0001 and 
                            abs(v[1] - vertex[1]) < 0.0001 and 
                            abs(v[2] - vertex[2]) < 0.0001):
                            face_indices.append(i)
                            break
                if face_indices:
                    faces.append(face_indices)
            
            mesh.from_pydata(vertices, [], faces)
        else:
            mesh.from_pydata(vertices, [], [])
    
    mesh.update()
    
    # Установка трансформаций
    obj.location = tuple(model_data["location"])
    obj.rotation_euler = tuple(model_data["rotation"])
    obj.scale = tuple(model_data["scale"])
    
    print(f"Модель '{model_data['name']}' загружена из файла: {filepath}")
    return obj

# Пример использования
filepath = os.path.join(bpy.path.abspath("//"), "exported_model.txt")
if os.path.exists(filepath):
    imported_obj = import_model_from_text(filepath)
else:
    print("Файл не найден")
```

4. Скрипт для создания окружностей с масштабированными моделями

```python
import bpy
import math
import json
import os

def clear_objects_by_name(prefix):
    """Удаляет объекты с определенным префиксом"""
    for obj in bpy.data.objects:
        if obj.name.startswith(prefix):
            bpy.data.objects.remove(obj, do_unlink=True)

def create_circular_pattern(model_data, circles=3, step_angle=15, base_radius=5):
    """Создает копии модели на окружностях с уменьшающимся масштабом"""
    
    # Очищаем предыдущие копии
    clear_objects_by_name("Copied_")
    
    for circle_idx in range(circles):
        # Радиус окружности увеличивается
        radius = base_radius * (circle_idx + 1)
        # Масштаб уменьшается с увеличением радиуса
        scale_factor = 1.0 / (circle_idx + 1)
        
        # Количество объектов на окружности
        num_objects = int(360 / step_angle)
        
        for i in range(num_objects):
            angle = math.radians(i * step_angle)
            
            # Вычисление позиции на окружности
            x = radius * math.cos(angle)
            y = radius * math.sin(angle)
            z = 0
            
            # Создание копии модели
            obj = create_model_from_data(model_data, f"Copied_{circle_idx+1}_{i+1}")
            
            # Установка позиции и масштаба
            obj.location = (x, y, z)
            obj.scale = (scale_factor, scale_factor, scale_factor)
            
            # Поворот объекта к центру
            direction = math.atan2(-y, -x)
            obj.rotation_euler = (0, 0, direction)

def create_model_from_data(model_data, name_suffix):
    """Создает модель из данных"""
    
    mesh = bpy.data.meshes.new(f"Mesh_{name_suffix}")
    obj = bpy.data.objects.new(name_suffix, mesh)
    bpy.context.collection.objects.link(obj)
    
    if model_data["vertices"]:
        vertices = [tuple(v) for v in model_data["vertices"]]
        faces = []
        
        if model_data["faces"]:
            for face in model_data["faces"]:
                face_indices = []
                for vertex in face:
                    for i, v in enumerate(vertices):
                        if (abs(v[0] - vertex[0]) < 0.0001 and 
                            abs(v[1] - vertex[1]) < 0.0001 and 
                            abs(v[2] - vertex[2]) < 0.0001):
                            face_indices.append(i)
                            break
                if face_indices:
                    faces.append(face_indices)
            
            mesh.from_pydata(vertices, [], faces)
        else:
            mesh.from_pydata(vertices, [], [])
    
    mesh.update()
    
    # Копируем материал если есть
    if model_data.get("material"):
        pass
    
    return obj

# Основная часть
filepath = os.path.join(bpy.path.abspath("//"), "exported_model.txt")
if os.path.exists(filepath):
    # Загружаем модель из файла
    with open(filepath, 'r', encoding='utf-8') as f:
        model_data = json.load(f)
    
    # Создаем круговой паттерн
    create_circular_pattern(model_data, circles=3, step_angle=30, base_radius=6)
    print("Создан круговой паттерн с масштабированными копиями")
else:
    print("Сначала экспортируйте модель с помощью скрипта №2")
```

5. Сцена с использованием БЯМ (Blender Yaml Manager)

```python
import bpy
import math
import random

def create_complex_scene():
    """Создает сложную сцену с использованием различных техник"""
    
    # Очистка сцены
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete(use_global=False)
    
    # 1. Создание сложной модели - Спиральная башня
    def create_spiral_tower():
        # Создание основной колонны
        bpy.ops.mesh.primitive_cylinder_add(vertices=32, radius=1.5, depth=8, location=(0, 0, 4))
        tower_base = bpy.context.active_object
        tower_base.name = "Tower_Base"
        
        # Создание спиральной лестницы
        for i in range(24):
            angle = i * math.pi / 12
            x = 2.5 * math.cos(angle)
            y = 2.5 * math.sin(angle)
            z = 0.3 * i
            
            bpy.ops.mesh.primitive_cube_add(size=0.4, location=(x, y, z))
            step = bpy.context.active_object
            step.name = f"Spiral_Step_{i}"
            step.scale = (1, 0.6, 0.2)
            
            # Добавление материала
            mat = bpy.data.materials.new(name=f"Step_Material_{i}")
            mat.use_nodes = True
            mat.node_tree.nodes["Principled BSDF"].inputs[0].default_value = (0.6, 0.4, 0.2, 1.0)
            step.data.materials.append(mat)
        
        # Создание шпиля
        bpy.ops.mesh.primitive_cone_add(radius1=1, radius2=0, depth=2, location=(0, 0, 8.5))
        spire = bpy.context.active_object
        spire.name = "Tower_Spire"
        
        # Золотой материал для шпиля
        gold_mat = bpy.data.materials.new(name="Gold_Material")
        gold_mat.use_nodes = True
        gold_mat.node_tree.nodes["Principled BSDF"].inputs[0].default_value = (0.8, 0.6, 0.1, 1.0)
        gold_mat.node_tree.nodes["Principled BSDF"].inputs[5].default_value = 1.0  # Металлик
        spire.data.materials.append(gold_mat)
        
        return tower_base
    
    # 2. Создание сложной модели - Кристаллическая структура
    def create_crystal_structure():
        crystals = []
        
        # Создание центрального кристалла
        bpy.ops.mesh.primitive_ico_sphere_add(subdivisions=2, radius=1.2, location=(5, 3, 1.5))
        center_crystal = bpy.context.active_object
        center_crystal.name = "Center_Crystal"
        
        # Создание окружающих кристаллов
        positions = [
            (6.5, 3.5, 2.2), (3.5, 3.5, 2.2), (5, 1.5, 2.5), (5, 4.5, 2.5),
            (6, 2.8, 2.8), (4, 2.8, 2.8), (5.5, 3.8, 2.8), (4.5, 3.8, 2.8)
        ]
        
        for i, pos in enumerate(positions):
            bpy.ops.mesh.primitive_cone_add(radius1=0.6, radius2=0, depth=1.5, location=pos)
            crystal = bpy.context.active_object
            crystal.name = f"Crystal_{i+1}"
            
            # Поворот к центру
            direction = math.atan2(pos[1] - 3, pos[0] - 5)
            crystal.rotation_euler = (0.3, 0.2, direction)
            
            # Материал для кристалла
            crystal_mat = bpy.data.materials.new(name=f"Crystal_Material_{i+1}")
            crystal_mat.use_nodes = True
            color = (random.random(), random.random(), random.random(), 1.0)
            crystal_mat.node_tree.nodes["Principled BSDF"].inputs[0].default_value = color
            crystal_mat.node_tree.nodes["Principled BSDF"].inputs[5].default_value = 0.8
            crystal_mat.node_tree.nodes["Principled BSDF"].inputs[6].default_value = 0.5  # Шероховатость
            crystal.data.materials.append(crystal_mat)
            
            crystals.append(crystal)
        
        return center_crystal, crystals
    
    # 3. Добавление освещения
    def add_lighting():
        # Основной свет
        bpy.ops.object.light_add(type='SUN', location=(10, 10, 15))
        sun_light = bpy.context.active_object
        sun_light.data.energy = 2
        
        # Заполняющий свет
        bpy.ops.object.light_add(type='AREA', location=(-5, -5, 5))
        fill_light = bpy.context.active_object
        fill_light.data.energy = 0.5
        
        # Цветной акцентный свет
        bpy.ops.object.light_add(type='POINT', location=(3, 4, 6))
        accent_light = bpy.context.active_object
        accent_light.data.energy = 1
        accent_light.data.color = (0.3, 0.5, 1.0)
    
    # 4. Добавление земли/подиума
    def add_ground():
        bpy.ops.mesh.primitive_plane_add(size=20, location=(0, 0, -0.5))
        ground = bpy.context.active_object
        ground.name = "Ground"
        
        # Материал земли
        ground_mat = bpy.data.materials.new(name="Ground_Material")
        ground_mat.use_nodes = True
        ground_mat.node_tree.nodes["Principled BSDF"].inputs[0].default_value = (0.2, 0.4, 0.1, 1.0)
        ground_mat.node_tree.nodes["Principled BSDF"].inputs[5].default_value = 0.1
        ground.data.materials.append(ground_mat)
    
    # 5. Добавление декоративных элементов
    def add_decorations():
        # Создание парящих сфер вокруг башни
        for i in range(16):
            angle = i * math.pi / 8
            x = 3.5 * math.cos(angle)
            y = 3.5 * math.sin(angle)
            z = 2 + math.sin(angle * 2) * 1.5
            
            bpy.ops.mesh.primitive_uv_sphere_add(radius=0.2, location=(x, y, z))
            sphere = bpy.context.active_object
            sphere.name = f"Floating_Sphere_{i}"
            
            # Эмиссивный материал
            emit_mat = bpy.data.materials.new(name=f"Emit_Material_{i}")
            emit_mat.use_nodes = True
            emit_mat.node_tree.nodes["Principled BSDF"].inputs[0].default_value = (1, 0.5, 0.2, 1.0)
            emit_mat.node_tree.nodes["Principled BSDF"].inputs[18].default_value = 0.8  # Эмиссия
            sphere.data.materials.append(emit_mat)
    
    # Создание сцены
    print("Создание сложной сцены...")
    create_spiral_tower()
    create_crystal_structure()
    add_lighting()
    add_ground()
    add_decorations()
    
    # Настройка камеры
    bpy.ops.object.camera_add(location=(12, -12, 8))
    camera = bpy.context.active_object
    camera.rotation_euler = (math.radians(60), 0, math.radians(45))
    bpy.context.scene.camera = camera
    
    print("Сцена успешно создана!")

# Запуск создания сцены
create_complex_scene()
```

Инструкция по использованию:

1. Для скрипта №1: Запустите в Blender (Scripting workspace), чтобы создать 10 случайных мешей с анимацией к точке (100, 100, 0).
2. Для скрипта №2: Выберите объект в сцене и запустите скрипт для сохранения его в текстовый файл.
3. Для скрипта №3: Запустите для загрузки модели из сохраненного файла.
4. Для скрипта №4: Запустите после выполнения скрипта №2, чтобы создать круговой паттерн с масштабированными копиями.
5. Для скрипта №5: Запустите для создания сложной сцены со спиральной башней и кристаллической структурой.

Все скрипты можно запускать в Blender через текстовый редактор, нажав кнопку "Run Script".
