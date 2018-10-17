# codingame
import sys
import math
import random
 
# Send your busters out into the fog to trap ghosts and bring them home!
 
def hunt_area(number_of_busters):
    tmp = int(16000/number_of_busters)
    result = [1000]
    for i in range(number_of_busters):
        result.append(tmp*(i+1))
    result.append(14500)
    return result
 
ghosts = []
busters = []
initiate_area = 0
add_busters = 0
vmovetmp = 0
vmove = [1950, 3900, 5850, 7000]
horizontal = 0
vertical = 0
afk = 0
 
class Ghost:
 
    def __init__(self, id, x, y, state, value):
        self.id = id
        self.x = x
        self.y = y
        self.state = state
        self.value = value
        self.target = False
 
class Buster:
 
    def __init__(self, id, x, y, left, right, state, value):
        self.id = id
        self.x = x
        self.y = y
        self.left = left
        self.right = right
        self.state = state
        self.catching = False
        self.destx = 0
        self.desty = 0
        self.chasing = [False, -1]
 
    def move(self):
        print("MOVE", self.destx, self.desty)
 
    def base(self, team_id):
        self.destx = team_id * 16000
        self.desty = team_id * 9000
 
    def bust(self, ghost_id):
        print("BUST", ghost_id)
 
    def release_ghost(self):
        print("RELEASE")
        self.catching = False
        self.chasing = [False, -1]
 
busters_per_player = int(input())  # the amount of busters you control
ghost_count = int(input())  # the amount of ghosts on the map
my_team_id = int(input())  # if this is 0, your base is on the top left of the map, if it is one, on the bottom right
hunt_area = hunt_area(busters_per_player)  #divides space to create areas for each buster to patrol
 
# game loop
while True:
    entities = int(input())  # the number of busters and ghosts visible to you
    for i in range(entities):
        # entity_id: buster id or ghost id
        # y: position of this buster / ghost
        # entity_type: the team id if it is a buster, -1 if it is a ghost.
        # state: For busters: 0=idle, 1=carrying a ghost.
        # value: For busters: Ghost id being carried. For ghosts: number of busters attempting to trap this ghost.
        entity_id, x, y, entity_type, state, value = [int(j) for j in input().split()]
        if entity_type == -1:
            add = True
            for i in ghosts:
                if i.id == entity_id:
                    add = False
            if add:
                ghosts.append(Ghost(entity_id, x, y, state, value))
            else:
                for i in ghosts:
                    if i.id == entity_id:
                        i.x = x
                        i.y = y
                        i.state = state
                        i.value = value
        elif add_busters < busters_per_player and entity_type == my_team_id:
            busters.append(Buster(entity_id, x, y, -1, -1, state, value))
            add_busters += 1
        elif entity_type == my_team_id:
            for i in busters:
                if i.id == entity_id:
                    i.state = state
                    i.value = value
                    i.x = x
                    i.y = y
 
    for i in range(busters_per_player):
        print("================", file=sys.stderr) 
        print(i, file=sys.stderr) 
 
        # Write an action using print
        # To debug: print("Debug messages...", file=sys.stderr)
 
        # MOVE x y | BUST id | RELEASE
        # print("MOVE 8000 4500")
 
        current_buster = busters[i]
 
        if initiate_area < busters_per_player:
                current_buster.left = hunt_area[i]
                current_buster.right = hunt_area[i+1]
                initiate_area += 1
                horizontal = random.randint(current_buster.left, current_buster.right)
                vertical = random.randint(vmove[vmovetmp], vmove[vmovetmp+1])
                current_buster.destx = horizontal
                current_buster.desty = vertical
 
        if current_buster.state:
            print("x1", file=sys.stderr) 
            current_buster.base(my_team_id)
            if abs(current_buster.x - current_buster.destx) < 120 and abs(current_buster.y - current_buster.desty) < 120:
                current_buster.release_ghost()
            else:
                current_buster.move()
        elif current_buster.chasing[0]:
            print("x2", file=sys.stderr) 
            print([current_buster.x, current_buster.y, current_buster.state, current_buster.catching, current_buster.destx, current_buster.desty, current_buster.chasing], file=sys.stderr)
            tmp = 0
            for i in ghosts:
                print([i.id, i.x, i.y, i.state, i.value], file=sys.stderr)
                tmp += 1
                if i.id == current_buster.chasing[1]:
                    sideL = math.sqrt((i.x - current_buster.x)**2 + (i.y - current_buster.y)**2)
                    if sideL < 1760:
                        if sideL > 900:
                            current_buster.catching = True
                            current_buster.bust(i.id)
                            break
                        else:
                            current_buster.move()
                            break
                    else:
                        current_buster.move()
                        break
            if tmp == len(ghosts):
                current_buster.chasing = [False, 0]
        else:
            print("x3", file=sys.stderr) 
            print([current_buster.x, current_buster.y, current_buster.state, current_buster.catching, current_buster.destx, current_buster.desty, current_buster.chasing], file=sys.stderr)
            for ghost in ghosts:
                print([ghost.id, ghost.x, ghost.y, ghost.state, ghost.value, ghost.target], file=sys.stderr) 
                #if ghost.x > current_buster.left and ghost.x < current_buster.right and not ghost.inbase and not ghost.target:
                if not ghost.target:
                       sidex = current_buster.x - ghost.x
                       sidey = current_buster.y - ghost.y
                       sideL = math.sqrt(sidex**2 + sidey**2)
                       if sideL < 3000:
                           if sideL < 900:
                               current_buster.destx = int(ghost.x + math.copysign(1000, sidex))
                               current_buster.desty = int(ghost.y + math.copysign(1000, sidey))
                               ghost.target = True
                               current_buster.chasing = [True, ghost.id]
                               print(current_buster.id, "move1", file=sys.stderr) 
                               current_buster.move()
                               break
                           elif math.sqrt((ghost.x - current_buster.x)**2 + (ghost.y - current_buster.y)**2) > 1700:
                               current_buster.destx = int(ghost.x + math.copysign(1000, sidex))
                               current_buster.desty = int(ghost.y + math.copysign(1000, sidey))
                               ghost.target = True
                               current_buster.chasing = [True, ghost.id]
                               print(current_buster.id, "move2", file=sys.stderr) 
                               current_buster.move()
                               break
                           else:
                               print(current_buster.id, "move3", file=sys.stderr) 
                               current_buster.catching = True
                               current_buster.bust(ghost.id)
                               break
                '''
                    ghost.target = True
                    current_buster.catching = True
                    current_buster.bust(ghost.id)
                    break
                '''
           
            if not current_buster.catching:
                if current_buster.x == current_buster.destx and current_buster.y == current_buster.desty:
                    vmovetmp = (vmovetmp + 1) % 3
                    horizontal = random.randint(current_buster.left, current_buster.right)
                    vertical = random.randint(vmove[vmovetmp], vmove[vmovetmp+1])
                    current_buster.destx = horizontal
                    current_buster.desty = vertical
                current_buster.move()
