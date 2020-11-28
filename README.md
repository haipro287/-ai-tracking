# ai-tracking
ai project 2020

## q1: Exact Inference Observation
    def observe(self, observation, gameState):
        noisyDistance = observation
        emissionModel = busters.getObservationDistribution(noisyDistance)
        pacmanPosition = gameState.getPacmanPosition()

        "*** YOUR CODE HERE ***"
        
        allPossible = util.Counter()
        
        // kiểm tra nếu có ma còn sống:
        if noisyDistance != None:
        
            // duyệt tất cả vị trí có thể tồn tại ma
            for p in self.legalPositions:
                        
                //  tìm khoảng cách tới vị trí đó
                trueDistance = util.manhattanDistance(p, pacmanPosition)
                
                // kiêm tra nếu phân bố của vị trí đó nếu > 0 thì cập nhật xác suất của vị trí
                if emissionModel[trueDistance] >0:
                    allPossible[p] = emissionModel[trueDistance] * self.beliefs[p]
        
        // nếu không thì gán xác suất của vị trí ma bị ăn bằng 1
        else:
            allPossible[self.getJailPosition()] = 1.0
        
        "*** END YOUR CODE HERE ***"

        allPossible.normalize()
        self.beliefs = allPossible
        

## q2: Exact Inference with Time Elapse
    def elapseTime(self, gameState):
        allPossible = util.Counter()
        
        // duyệt tất cả vị trí có thể tồn tại ma
        for oldPos in self.legalPositions:
        
            // lấy phân bố tất cả vị trí ma có thể đi đến trong bước tiếp theo
            newPosDist = self.getPositionDistribution(self.setGhostPosition(gameState, oldPos))
            
            // duyệt các vị trí mới
            for newPos, probability in newPosDist.items():
            
                // tính xác suất của tất cả vị trí mới
                allPossible[newPos] += probability * self.beliefs[oldPos]
                
        self.beliefs = allPossible

## q3: Exact Inference Full Test
    
    def chooseAction(self, gameState):
        pacmanPosition = gameState.getPacmanPosition()
        legal = [a for a in gameState.getLegalPacmanActions()]
        livingGhosts = gameState.getLivingGhosts()
        livingGhostPositionDistributions = \
            [beliefs for i, beliefs in enumerate(self.ghostBeliefs)
             if livingGhosts[i+1]]
        "*** YOUR CODE HERE ***"
        
        // tìm các xác suất cao nhất đối với mỗi ma còn sống
        livingGhostCurrentPosition = []
        for GhostPositionDistributions in livingGhostPositionDistributions:
            currentPosition = GhostPositionDistributions.argMax()
            livingGhostCurrentPosition.append(currentPosition)

        // lấy khoảng cách tới tất cả vị trí vừa tìm
        livingGhostDistance = []
        for currentPosition in livingGhostCurrentPosition:
            distance = self.distancer.getDistance(pacmanPosition, currentPosition)
            livingGhostDistance.append(distance)

        // tính khoảng cách tới ma gần nhất vừa tìm 
        nearestGhostDistance = min(livingGhostDistance)
        nearestGhostPosition = livingGhostCurrentPosition[livingGhostDistance.index(nearestGhostDistance)]

        // chọn ra hành động tốt nhất để đến vị trí gần nhất vừa tìm
        actions = []
        for action in legal:
            nextPosition = Actions.getSuccessor(pacmanPosition, action)
            actions.append((self.distancer.getDistance(nextPosition, nearestGhostPosition), action))            
        return  min(actions)[1]
        
## q4: Approximate Inference Observation

    def initializeUniformly(self, gameState):
        "*** YOUR CODE HERE ***"
        
        self.particles = []
        
        // gán các vị trí hợp lệ vào các particles
        for i in range(self.numParticles):
            for position in self.legalPositions:
                self.particles.append(position)
                if len(self.particles) >= self.numParticles:
                    return
    
    def observe()
        noisyDistance = observation
        emissionModel = busters.getObservationDistribution(noisyDistance)
        pacmanPosition = gameState.getPacmanPosition()
        "*** YOUR CODE HERE ***"
        
        // kiểm tra nếu không ma còn sống:
        if noisyDistance == None:
            self.particles = [self.getJailPosition() for i in range(self.numParticles)]
            return
        
        // nếu vẫn còn ma sống, tìm khoảng cách và tính xác suất của các vị trí hợp lệ
        allPossible = util.Counter()
        beliefs = self.getBeliefDistribution()
        for p in self.legalPositions:
            trueDistance = util.manhattanDistance(p, pacmanPosition)
            if emissionModel[trueDistance] > 0:
                allPossible[p] = beliefs[p] * emissionModel[trueDistance]
                
        // nếu không có vị trí naò hợp lệ thì reset lại, nếu không gán lại các particles
        if allPossible.totalCount() == 0:
            self.initializeUniformly(gameState)
        else:
            self.particles = [util.sample(allPossible) for i in range(self.numParticles)]
            
   def getBeliefDistribution(self):
        "*** YOUR CODE HERE ***"
        
        beliefs = util.Counter()
        // tăng các giá trị xác suất thêm 1
        for p in self.particles:
            beliefs[p] += 1.0
        
        // đưa lại các giá trị về tổng = 1
        beliefs.normalize()
        return beliefs
            
## q5: Approximate Inference with Time Elapse
    
    def elapseTime(self, gameState):
        "*** YOUR CODE HERE ***"

        newParticles = []
        
        // tìm các particles mới từ các vị trí hợp lệ hiện tại
        for p in self.particles:
            newPositionDistribution = self.getPositionDistribution(self.setGhostPosition(gameState, p))
            newParticles.append(util.sample(newPositionDistribution))
        self.particles = newParticles
        

    
