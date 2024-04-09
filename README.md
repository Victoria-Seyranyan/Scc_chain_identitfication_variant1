# Scc_chain_identitfication_variant1
#include <iostream>
#include <vector>
#include <unordered_map>
#include <unordered_set>
#include <queue>
#include <algorithm>



class StageMapper {
private:
    std::unordered_map<std::string, int> name_to_index;
    std::unordered_map<int, std::string> index_to_name;

public:
    void addStage(const std::string& stage) {
        int index = name_to_index.size();
        name_to_index[stage] = index;
        index_to_name[index] = stage;
    }

    int getIndex(const std::string& stage) const {
        return name_to_index.at(stage);
    }

    std::string getName(int index) const {
        return index_to_name.at(index);
    }

    int size() const {
        return name_to_index.size();
    }
};

class Production {
private:
   std::vector<std::vector<int>> transitions;
    std::unordered_set<int> irreversibleStages;

public:
    Production(int n) : transitions(n) {}

    void addTransition(int from, int to) {
        transitions[from].push_back(to);
    }

    void markIrreversible(int stage) {
        irreversibleStages.insert(stage);
    }

    bool isIrreversible(int stage) const {
        return irreversibleStages.count(stage);
    }

    const std::vector<int>& getTransitions(int stage) const {
        return transitions[stage];
    }

    int size() const {
        return transitions.size();
    }
};

class ProductionAnalyzer {
private:
    const Production& production;
    const StageMapper& mapper;

public:
    ProductionAnalyzer(const Production& p, const StageMapper& m) : production(p), mapper(m) {}

    void analyze() const {
        printIrreversibleStages();
        printReachableStages();
        printProductionLinearity();
    }

private:
    void printIrreversibleStages() const {
       std:: cout << "Non-reversible stages:" << std::endl;
        for (int stage = 0; stage < production.size(); ++stage) {
            if (production.isIrreversible(stage)) {
                std::cout << mapper.getName(stage) << std::endl;
            }
        }
    }

    void printReachableStages() const {
        std::cout << "Stages that can be reached:" << std::endl;
        for (int stage = 0; stage < production.size(); ++stage) {
            std::cout << "From stage " << mapper.getName(stage) << ", we can go to: ";
            for (int nextStage : production.getTransitions(stage)) {
               std:: cout << mapper.getName(nextStage) << " ";
            }
            std::cout << std::endl;
        }
    }

    void printProductionLinearity() const {
        std::vector<int> linearOrder = topologicalSort();
        std::cout << "Is the production linear? ";
        if (linearOrder.empty()) {
            std::cout << "No" << std::endl;
        }
        else {
            std::cout << "Yes" << std::endl;
            std::cout << "Linear order: ";
            for (int stage : linearOrder) {
                std::cout << mapper.getName(stage) << " ";
            }
            std::cout << std::endl;
        }
    }

    std::vector<int> topologicalSort() const {
        std::vector<int> inDegree(production.size(), 0);
        for (int stage = 0; stage < production.size(); ++stage) {
            for (int nextStage : production.getTransitions(stage)) {
                ++inDegree[nextStage];
            }
        }

        std::queue<int> q;
        for (int stage = 0; stage < production.size(); ++stage) {
            if (inDegree[stage] == 0) {
                q.push(stage);
            }
        }

        std::vector<int> linearOrder;
        while (!q.empty()) {
            int stage = q.front();
            q.pop();
            linearOrder.push_back(stage);
            for (int nextStage : production.getTransitions(stage)) {
                if (--inDegree[nextStage] == 0) {
                    q.push(nextStage);
                }
            }
        }

        if (linearOrder.size() != production.size()) {
            return {}; // Production is not linear
        }
        return linearOrder;
    }
};

int main() {

    int n = 4; // Number of stages
    std::vector<std::string> stages = { "A", "B", "C", "D" };
    int m = 6; // Number of transitions
    std::vector<std::pair<std::string, std::string>> transitions = { {"A", "B"}, {"A", "C"}, {"B", "D"}, {"C", "D"}, {"D", "C"}, {"B", "A"} };
    int l = 2; // Number of non-reversible stages
    std::vector<std::string> nonReversibleStages = { "A", "D" };

   
    StageMapper mapper;
    for (const std::string& stage : stages) {
        mapper.addStage(stage);
    }

    
    Production production(n);
    for (const auto& transition : transitions) {
        int fromIndex = mapper.getIndex(transition.first);
        int toIndex = mapper.getIndex(transition.second);
        production.addTransition(fromIndex, toIndex);
    }

  
    for (const std::string& stage : nonReversibleStages) {
        int stageIndex = mapper.getIndex(stage);
        production.markIrreversible(stageIndex);
    }

   
    ProductionAnalyzer analyzer(production, mapper);
    analyzer.analyze();

    return 0;
}
