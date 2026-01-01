# Genetic Algorithm

## 1. Overview

Genetic algorithms (GAs) are evolutionary optimization techniques inspired by natural selection and genetics. Developed by John Holland in the 1960s and popularized in the 1970s, GAs use mechanisms like selection, crossover, and mutation to evolve solutions to optimization and search problems.

GAs are particularly effective for complex, non-linear optimization problems where traditional gradient-based methods fail or where the search space is too large for exhaustive enumeration.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an objective function $f: S \rightarrow \mathbb{R}$ where $S$ is the search space, find $x^* \in S$ such that:

$$x^* = \arg\max_{x \in S} f(x)$$

(or minimize, depending on context)

### 2.2 Mathematical Model

**Components**:
1. **Population**: Set of candidate solutions $P = \{x_1, x_2, ..., x_n\}$
2. **Fitness Function**: $f: S \rightarrow \mathbb{R}$ evaluates solution quality
3. **Selection**: Probability of choosing individual $i$: $p_i = \frac{f(x_i)}{\sum_{j=1}^{n} f(x_j)}$ (fitness-proportional)
4. **Crossover**: Combine parents to create offspring
5. **Mutation**: Random modification with probability $p_m$

**Evolution Equation**: 
$$P_{t+1} = \text{Mutate}(\text{Crossover}(\text{Select}(P_t)))$$

### 2.3 Correctness (Schema Theorem)

**Holland's Schema Theorem**: Short, low-order, above-average schemata receive exponentially increasing trials in successive generations.

A **schema** is a template (pattern) of solutions. GAs implicitly evaluate $O(n^3)$ schemata with population of size $n$, providing massive implicit parallelism.

## 3. Algorithm Description

### 3.1 Intuition

Genetic algorithms mimic natural evolution:

1. **Initialize**: Create random population of solutions
2. **Evaluate**: Calculate fitness of each individual
3. **Select**: Choose parents based on fitness (survival of the fittest)
4. **Crossover**: Combine pairs of parents to create offspring
5. **Mutate**: Randomly alter some offspring
6. **Replace**: Form new population from offspring
7. **Repeat**: Steps 2-6 until convergence or max generations

**Why it works**: Good solutions produce better offspring, bad solutions die out, mutation maintains diversity and explores new areas.

### 3.2 Pseudocode

```
function GeneticAlgorithm(population_size, max_generations, mutation_rate):
    // Step 1: Initialize population
    population = generateRandomPopulation(population_size)
    
    for generation from 1 to max_generations:
        // Step 2: Evaluate fitness
        fitness_scores = [fitness(individual) for individual in population]
        
        // Check termination
        if max(fitness_scores) meets termination_criteria:
            return best individual
        
        // Step 3: Create new population
        new_population = []
        
        while size(new_population) < population_size:
            // Selection
            parent1 = selectParent(population, fitness_scores)
            parent2 = selectParent(population, fitness_scores)
            
            // Crossover
            child1, child2 = crossover(parent1, parent2)
            
            // Mutation
            child1 = mutate(child1, mutation_rate)
            child2 = mutate(child2, mutation_rate)
            
            new_population.append(child1)
            if size(new_population) < population_size:
                new_population.append(child2)
        
        // Step 4: Elitism (optional - keep best individuals)
        new_population = applyElitism(population, new_population, elite_size)
        
        population = new_population
    
    return best individual from population

function selectParent(population, fitness_scores):
    // Roulette wheel selection
    total_fitness = sum(fitness_scores)
    pick = random(0, total_fitness)
    current = 0
    
    for i from 0 to size(population) - 1:
        current += fitness_scores[i]
        if current > pick:
            return population[i]

// Alternative: Tournament selection
function selectParentTournament(population, fitness_scores, tournament_size):
    tournament = random sample of tournament_size individuals
    return best individual from tournament

function crossover(parent1, parent2):
    // Single-point crossover
    crossover_point = random(1, length(parent1) - 1)
    
    child1 = parent1[0:crossover_point] + parent2[crossover_point:end]
    child2 = parent2[0:crossover_point] + parent1[crossover_point:end]
    
    return (child1, child2)

function mutate(individual, mutation_rate):
    for i from 0 to length(individual) - 1:
        if random() < mutation_rate:
            individual[i] = randomValue()
    
    return individual
```

### 3.3 Step-by-Step Example

**Problem**: Maximize $f(x) = x^2$ where $x \in [0, 31]$ (5-bit binary encoding)

```
Generation 0: Initialize
  Individual 1: 01101 (13) → f(13) = 169
  Individual 2: 11000 (24) → f(24) = 576
  Individual 3: 01000 (8)  → f(8)  = 64
  Individual 4: 10011 (19) → f(19) = 361
  
  Total fitness = 1170
  Probabilities: 0.144, 0.492, 0.055, 0.309

Generation 1: Selection, Crossover, Mutation
  
  Selection (roulette):
    Parent 1: 11000 (picked based on high fitness)
    Parent 2: 10011
  
  Crossover at position 3:
    Parent 1: 110|00
    Parent 2: 100|11
    Child 1:  110|11 = 27 → f(27) = 729 ✓
    Child 2:  100|00 = 16 → f(16) = 256
  
  Mutation (rate = 0.01):
    Child 1: 11011 → no mutation
    Child 2: 10000 → mutate bit 3 → 10010 = 18, f(18) = 324

Continue for multiple generations...

Generation 10: Population converges near optimal
  Individual 1: 11111 (31) → f(31) = 961
  Individual 2: 11111 (31) → f(31) = 961
  Individual 3: 11110 (30) → f(30) = 900
  Individual 4: 11111 (31) → f(31) = 961
  
  Best solution: x = 31 (optimal)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Per generation**:
  - Fitness evaluation: $O(nf)$ where $n$ = population size, $f$ = fitness computation cost
  - Selection: $O(n)$ for roulette wheel, $O(t \log t)$ for tournament
  - Crossover: $O(nl)$ where $l$ = chromosome length
  - Mutation: $O(nl)$
  - Total per generation: $O(n(f + l))$

- **Total**: $O(Gn(f + l))$ where $G$ = number of generations

The dominant factor is usually fitness evaluation.

### 4.2 Space Complexity

- **Population storage**: $O(nl)$
- **Fitness scores**: $O(n)$
- **Total**: $O(nl)$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use rand::Rng;
use rand::seq::SliceRandom;

trait Individual: Clone {
    fn fitness(&self) -> f64;
    fn crossover(&self, other: &Self) -> (Self, Self);
    fn mutate(&mut self, rate: f64);
}

struct GeneticAlgorithm<T: Individual> {
    population: Vec<T>,
    mutation_rate: f64,
    elite_size: usize,
}

impl<T: Individual> GeneticAlgorithm<T> {
    fn evolve(&mut self, generations: usize) -> T {
        for _ in 0..generations {
            let fitness_scores: Vec<f64> = self.population
                .iter()
                .map(|ind| ind.fitness())
                .collect();
            
            let mut new_population = Vec::new();
            
            // Elitism
            let mut sorted_pop = self.population.clone();
            sorted_pop.sort_by(|a, b| {
                b.fitness().partial_cmp(&a.fitness()).unwrap()
            });
            
            for i in 0..self.elite_size {
                new_population.push(sorted_pop[i].clone());
            }
            
            // Generate offspring
            while new_population.len() < self.population.len() {
                let parent1 = self.select(&fitness_scores);
                let parent2 = self.select(&fitness_scores);
                
                let (mut child1, mut child2) = parent1.crossover(&parent2);
                
                child1.mutate(self.mutation_rate);
                child2.mutate(self.mutation_rate);
                
                new_population.push(child1);
                if new_population.len() < self.population.len() {
                    new_population.push(child2);
                }
            }
            
            self.population = new_population;
        }
        
        self.best_individual()
    }
    
    fn select(&self, fitness_scores: &[f64]) -> T {
        let total_fitness: f64 = fitness_scores.iter().sum();
        let mut rng = rand::thread_rng();
        let pick = rng.gen::<f64>() * total_fitness;
        
        let mut current = 0.0;
        for (i, &score) in fitness_scores.iter().enumerate() {
            current += score;
            if current > pick {
                return self.population[i].clone();
            }
        }
        
        self.population.last().unwrap().clone()
    }
    
    fn best_individual(&self) -> T {
        self.population
            .iter()
            .max_by(|a, b| {
                a.fitness().partial_cmp(&b.fitness()).unwrap()
            })
            .unwrap()
            .clone()
    }
}
```

### 5.2 Edge Cases

1. **Small population**: Premature convergence, loss of diversity
2. **Large population**: Slow convergence
3. **High mutation rate**: Random search behavior
4. **Low mutation rate**: Stuck in local optima
5. **No elitism**: Best solutions may be lost
6. **Fitness scaling**: Very different fitness values affect selection pressure

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Optimization Problems**
- Parameter tuning
- Neural network architecture search
- Hyperparameter optimization
- Resource allocation

**2. Scheduling**
- Job shop scheduling
- Timetabling
- Task assignment
- Route optimization

**3. Game Development**
- AI behavior evolution
- Procedural content generation
- Game balancing
- Strategy optimization

**4. Engineering Design**
- Circuit design
- Antenna design
- Structural optimization
- Control system design

**5. Machine Learning**
- Feature selection
- Ensemble method optimization
- Neural architecture search (NAS)

### 6.2 Related Algorithms

**Evolutionary Algorithms**:
- **Evolution Strategies**: Focus on real-valued optimization
- **Genetic Programming**: Evolves programs/trees
- **Differential Evolution**: Mutation based on population differences
- **Particle Swarm Optimization**: Swarm intelligence

**Hybrid Approaches**:
- **Memetic Algorithms**: GA + local search
- **NSGA-II**: Multi-objective optimization
- **Island Models**: Parallel GAs with migration

## 7. References

### Academic Papers
1. Holland, J.H. (1992). *Adaptation in Natural and Artificial Systems*. MIT Press.
2. Goldberg, D.E. (1989). *Genetic Algorithms in Search, Optimization, and Machine Learning*. Addison-Wesley.
3. De Jong, K.A. (1975). "Analysis of the Behavior of a Class of Genetic Adaptive Systems". PhD thesis, University of Michigan.

### Books
1. Mitchell, M. (1998). *An Introduction to Genetic Algorithms*. MIT Press.
2. Eiben, A.E., & Smith, J.E. (2015). *Introduction to Evolutionary Computing* (2nd ed.). Springer.

### Online Resources
1. [Genetic Algorithm - Wikipedia](https://en.wikipedia.org/wiki/Genetic_algorithm)
2. [GA Tutorial - MIT](https://web.mit.edu/6.034/wwwbob/gahandout.html)

### Implementation
- Source: `src/general/genetic.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
