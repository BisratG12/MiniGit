# MiniGit
```
#include <iostream>
#include <filesystem>
#include <unordered_map>
#include <vector>
namespace fs = std::filesystem;

// Simple hash using SHA-1 would require a library; here's a placeholder:
std::string simpleHash(const std::string& data) {
    std::hash<std::string> h;
    return std::to_string(h(data));
}

class MiniGit {
    fs::path repoPath, objectsPath, headPath, indexPath;

public:
    MiniGit(const std::string& base = ".") {
        repoPath   = fs::path(base) / ".minigit";
        objectsPath = repoPath / "objects";
        headPath    = repoPath / "HEAD";
        indexPath   = repoPath / "index";
        initRepo();
        std::cout << "Repo initialized at: " << repoPath << "\n";
    }

    void initRepo() {
        fs::create_directories(objectsPath);
        std::ofstream(headPath, std::ios::app);
        std::ofstream(indexPath, std::ios::trunc);
```
