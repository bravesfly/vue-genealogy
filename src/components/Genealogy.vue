<template>
  <div class="h-screen flex flex-col bg-background">
    <!-- Header 区域 - 移动端优化 -->
    <header class="bg-card/50 backdrop-blur-sm border-b flex-shrink-0">
      <!-- 移动端：单行布局 -->
      <div class="block md:hidden">
        <div class="px-4 py-3 flex items-center gap-3">
          <!-- Logo 占位区域 -->
          <img src="/logo.png" class="w-8 h-8" />

          <!-- 搜索框 -->
          <Input
            v-model="searchValue"
            placeholder="搜索..."
            class="flex-1 text-base min-w-0"
            @keyup.enter="handleSearchGenealogy"
          />

          <!-- 工具下拉框 -->
          <DropdownMenu>
            <DropdownMenuTrigger as-child>
              <Button variant="outline" size="sm" class="w-16 text-xs">
                工具
                <svg
                  class="w-3 h-3 ml-1"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M19 9l-7 7-7-7"
                  ></path>
                </svg>
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent align="end">
              <DropdownMenuItem @click="handleSelectTool('ancestor')">
                共同先祖
              </DropdownMenuItem>
              <DropdownMenuItem @click="handleSelectTool('level')">
                辈分计算
              </DropdownMenuItem>
              <DropdownMenuItem @click="handleSelectTool('eighteen')">
                祖宗十八代
              </DropdownMenuItem>
            </DropdownMenuContent>
          </DropdownMenu>

          <!-- 刷新按钮 -->
          <Button @click="refreshData" variant="outline" size="sm" class="p-2">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
              />
            </svg>
          </Button>
        </div>
      </div>

      <!-- 桌面端：水平布局 -->
      <div class="hidden md:block">
        <div class="h-16 px-6 flex items-center justify-between">
          <div class="flex items-center">
            <h1 class="text-xl font-semibold text-foreground">
              塔林戴氏电子联谱
            </h1>
          </div>

          <div class="flex items-center gap-3">
            <Input
              v-model="searchValue"
              placeholder="搜索族谱成员..."
              class="w-64"
              @keyup.enter="handleSearchGenealogy"
            />

            <DropdownMenu>
              <DropdownMenuTrigger as-child>
                <Button variant="outline" class="w-32">
                  选择工具
                  <svg
                    class="w-4 h-4 ml-1"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M19 9l-7 7-7-7"
                    ></path>
                  </svg>
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="start">
                <DropdownMenuItem @click="handleSelectTool('ancestor')">
                  共同先祖
                </DropdownMenuItem>
                <DropdownMenuItem @click="handleSelectTool('level')">
                  辈分计算
                </DropdownMenuItem>
                <DropdownMenuItem @click="handleSelectTool('eighteen')">
                  祖宗十八代
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>

            <Button @click="refreshData" variant="outline" size="sm">
              刷新数据
            </Button>
          </div>
        </div>
      </div>
    </header>

    <!-- 主内容区域 -->
    <main class="flex-1 min-h-0 overflow-hidden">
      <div class="w-full h-full relative">
        <!-- 加载状态 -->
        <div
          v-if="loading"
          class="absolute inset-0 flex items-center justify-center bg-background/80 backdrop-blur-sm z-10"
        >
          <div class="flex flex-col items-center space-y-4 px-4">
            <div
              class="animate-spin rounded-full h-8 w-8 border-b-2 border-primary"
            ></div>
            <p class="text-sm md:text-base text-muted-foreground text-center">
              正在加载族谱数据...
            </p>
          </div>
        </div>

        <!-- 错误状态 -->
        <div
          v-else-if="error"
          class="absolute inset-0 flex items-center justify-center p-4"
        >
          <div class="text-center space-y-4 max-w-sm">
            <div class="text-destructive text-lg font-medium">数据加载失败</div>
            <p class="text-muted-foreground text-sm">{{ error }}</p>
            <Button
              @click="fetchData"
              variant="outline"
              size="default"
              class="w-full md:w-auto"
            >
              重新加载
            </Button>
          </div>
        </div>

        <!-- G6 图谱容器 -->
        <div
          ref="graphContainer"
          id="mountNode"
          class="w-full h-full bg-background"
          :style="{
            display: loading || error ? 'none' : 'block',
            minHeight: '400px',
            minWidth: '300px',
          }"
        ></div>

        <!-- 移动端浮动工具栏 -->
        <div
          class="md:hidden absolute bottom-4 left-4 right-4"
          v-if="!loading && !error && graph"
        >
          <div
            class="bg-card/90 backdrop-blur-sm border rounded-lg p-3 shadow-lg"
          >
            <div class="flex justify-between items-center gap-2">
              <Button
                size="sm"
                variant="secondary"
                class="flex-1 text-xs"
                @click="resetView"
              >
                重置视图
              </Button>
              <Button
                size="sm"
                variant="secondary"
                class="flex-1 text-xs"
                @click="fitView"
              >
                适应画布
              </Button>
              <Button
                size="sm"
                variant="secondary"
                class="flex-1 text-xs"
                @click="downloadImage"
              >
                下载图片
              </Button>
            </div>
          </div>
        </div>

        <!-- 桌面端工具栏 -->
        <div
          class="hidden md:block absolute bottom-6 left-6"
          v-if="!loading && !error && graph"
        >
          <div class="flex space-x-2">
            <Button size="sm" variant="secondary" @click="resetView">
              重置视图
            </Button>
            <Button size="sm" variant="secondary" @click="fitView">
              适应画布
            </Button>
            <Button size="sm" variant="secondary" @click="downloadImage">
              下载图片
            </Button>
          </div>
        </div>
      </div>
    </main>

    <!-- 节点详情模态框 -->
    <div
      v-if="showNodeModal"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div
        class="bg-card rounded-lg shadow-xl max-w-2xl w-full max-h-[90vh] overflow-y-auto"
      >
        <div class="flex items-center justify-between p-4 md:p-6 border-b">
          <h2 class="text-xl font-semibold">
            {{ getTreeNodeLabel(modalData) }}
          </h2>
          <Button variant="ghost" size="sm" @click="showNodeModal = false">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-4 md:p-6 space-y-4 md:space-y-6" v-if="modalData.id">
          <!-- 基本信息 -->
          <div class="grid grid-cols-2 gap-4">
            <div class="space-y-2">
              <div class="text-sm text-muted-foreground">姓名</div>
              <div class="font-medium">{{ modalData.name || "无" }}</div>
            </div>
            <div class="space-y-2">
              <div class="text-sm text-muted-foreground">字</div>
              <div class="font-medium">{{ modalData.style_name || "无" }}</div>
            </div>
            <div class="space-y-2" v-if="modalData.posthumous_title">
              <div class="text-sm text-muted-foreground">谥号</div>
              <div class="font-medium">{{ modalData.posthumous_title }}</div>
            </div>
            <div class="space-y-2">
              <div class="text-sm text-muted-foreground">性别</div>
              <div class="font-medium">
                <span
                  :class="
                    modalData.gender === 1 ? 'text-blue-600' : 'text-pink-600'
                  "
                >
                  {{ modalData.gender === 1 ? "男" : "女" }}
                </span>
              </div>
            </div>
            <div class="space-y-2">
              <div class="text-sm text-muted-foreground">世系</div>
              <div class="font-medium">{{ modalData.lineage || "无" }}</div>
            </div>
            <div class="space-y-2" v-if="modalData.style_name_level">
              <div class="text-sm text-muted-foreground">字辈</div>
              <div class="font-medium">{{ modalData.style_name_level }}</div>
            </div>
            <div class="space-y-2" v-if="modalData.faction">
              <div class="text-sm text-muted-foreground">房派</div>
              <div class="font-medium">{{ modalData.faction }}</div>
            </div>
            <div class="space-y-2" v-if="modalData.status">
              <div class="text-sm text-muted-foreground">状态</div>
              <div class="font-medium">{{ modalData.status }}</div>
            </div>
          </div>

          <!-- 生卒年信息 -->
          <div
            v-if="
              modalData.lunar_birthday_date ||
              modalData.lunar_death_date ||
              modalData.birthday_date ||
              modalData.death_date
            "
            class="space-y-4"
          >
            <h3 class="text-lg font-semibold border-b pb-2 mb-3 md:mb-4">
              生卒年月
            </h3>
            <div class="grid grid-cols-2 gap-4">
              <div
                class="space-y-2"
                v-if="modalData.lunar_birthday_date || modalData.birthday_date"
              >
                <div class="text-sm text-muted-foreground">出生日期</div>
                <div class="font-medium text-green-600">
                  {{
                    formatBirthDeathDate(
                      modalData.lunar_birthday_date || modalData.birthday_date
                    )
                  }}
                </div>
              </div>
              <div
                class="space-y-2"
                v-if="modalData.lunar_death_date || modalData.death_date"
              >
                <div class="text-sm text-muted-foreground">逝世日期</div>
                <div class="font-medium text-red-600">
                  {{
                    formatBirthDeathDate(
                      modalData.lunar_death_date || modalData.death_date
                    )
                  }}
                </div>
              </div>
            </div>
          </div>

          <!-- 描述信息 -->
          <div class="space-y-2" v-if="modalData.desc">
            <h3 class="text-lg font-semibold border-b pb-2 mb-3 md:mb-4">
              个人描述
            </h3>
            <div class="text-sm leading-relaxed">{{ modalData.desc }}</div>
          </div>

          <!-- 配偶信息 -->
          <div
            class="space-y-3 md:space-y-4"
            v-if="modalData.spouses && modalData.spouses.length > 0"
          >
            <h3 class="text-lg font-semibold border-b pb-2 mb-3 md:mb-4">
              配偶信息
            </h3>
            <div class="space-y-2 md:space-y-3">
              <div
                v-for="(spouse, index) in modalData.spouses"
                :key="index"
                                 class="p-3 md:p-4 bg-gray-50 rounded-lg border border-gray-200"
              >
                <div class="flex items-start justify-between mb-3">
                  <div>
                    <div class="font-medium text-lg">
                      {{ spouse.spouse.name || "无名" }}
                    </div>
                    <div class="text-sm text-muted-foreground">
                                             <span
                         class="inline-flex items-center px-2 py-1 rounded-full text-xs bg-blue-100 text-blue-800 border border-blue-200"
                         style="color: #1e40af !important; background-color: #dbeafe !important;"
                       >
                         {{ spouse.role || "配偶" }}
                       </span>
                    </div>
                  </div>
                  <div class="text-sm text-muted-foreground">
                    第{{ index + 1 }}位配偶
                  </div>
                </div>

                <!-- 配偶生卒年 -->
                <div
                  v-if="
                    spouse.spouse.lunar_birthday_date ||
                    spouse.spouse.lunar_death_date ||
                    spouse.spouse.birthday_date ||
                    spouse.spouse.death_date
                  "
                  class="grid grid-cols-2 gap-3 mt-3 pt-3 border-t border-muted"
                >
                  <div
                    class="space-y-1"
                    v-if="
                      spouse.spouse.lunar_birthday_date ||
                      spouse.spouse.birthday_date
                    "
                  >
                    <div class="text-xs text-muted-foreground">出生</div>
                    <div class="text-sm text-green-600">
                      {{
                        formatBirthDeathDate(
                          spouse.spouse.lunar_birthday_date ||
                            spouse.spouse.birthday_date
                        )
                      }}
                    </div>
                  </div>
                  <div
                    class="space-y-1"
                    v-if="
                      spouse.spouse.lunar_death_date || spouse.spouse.death_date
                    "
                  >
                    <div class="text-xs text-muted-foreground">逝世</div>
                    <div class="text-sm text-red-600">
                      {{
                        formatBirthDeathDate(
                          spouse.spouse.lunar_death_date ||
                            spouse.spouse.death_date
                        )
                      }}
                    </div>
                  </div>
                </div>

                <!-- 配偶其他信息 -->
                <div
                  v-if="spouse.spouse.desc"
                  class="mt-3 pt-3 border-t border-muted"
                >
                  <div class="text-xs text-muted-foreground mb-1">备注</div>
                  <div class="text-sm text-muted-foreground">
                    {{ spouse.spouse.desc }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 搜索结果模态框 -->
    <div
      v-if="showSearchResult"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-card rounded-lg shadow-xl max-w-md w-full max-h-[80vh]">
        <div class="flex items-center justify-between p-4 border-b">
          <h2 class="text-lg font-semibold">搜索结果</h2>
          <Button variant="ghost" size="sm" @click="showSearchResult = false">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-4">
          <div v-if="searchResult.length === 0" class="text-center py-8">
            <p class="text-muted-foreground">没有找到结果</p>
            <Button class="mt-4" @click="showSearchResult = false"
              >重新搜索</Button
            >
          </div>
          <div v-else class="space-y-2 max-h-96 overflow-y-auto">
            <div
              v-for="item in searchResult"
              :key="item.id"
              @click="handleSearchResultClick(item.id)"
              class="p-3 hover:bg-muted/50 rounded-lg cursor-pointer transition-colors"
            >
              <div class="font-medium">{{ getGenealogyName(item) }}</div>
              <div class="text-sm text-muted-foreground">
                {{ item.gender === 1 ? "男" : "女" }} · 世系: {{ item.lineage }}
              </div>
              <div class="text-xs text-muted-foreground mt-1 line-clamp-2">
                {{ item.desc }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 共同先祖搜索弹窗 -->
    <div
      v-if="showAncestorModal"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div
        class="bg-card rounded-lg shadow-xl max-w-md w-full max-h-[90vh] overflow-y-auto"
      >
        <div class="flex items-center justify-between p-4 border-b">
          <h2 class="text-lg font-semibold">查找共同先祖</h2>
          <Button variant="ghost" size="sm" @click="resetAncestorTool">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-4 space-y-4">
          <div class="text-sm text-muted-foreground mb-4">
            请输入姓名搜索，然后从下拉列表中选择准确的人物
          </div>

          <!-- 第一个人搜索 -->
          <div class="space-y-2">
            <label class="text-sm font-medium">第一个人</label>
            <div class="flex space-x-2">
              <div class="relative flex-1">
                <Input
                  v-model="ancestorSearchValue1"
                  placeholder="输入姓名搜索..."
                  @input="onInput1Change"
                  @keyup.enter="handleSearchPerson1"
                  :class="selectedPerson1 ? 'border-green-500' : ''"
                />
                <div
                  v-if="ancestorSearchResults1.length > 0"
                  class="absolute top-full left-0 right-0 bg-card border rounded-md shadow-lg z-10 max-h-40 overflow-y-auto"
                >
                  <div class="px-2 py-1 text-xs text-muted-foreground border-b">
                    找到 {{ ancestorSearchResults1.length }} 个结果，点击选择
                  </div>
                  <div
                    v-for="person in ancestorSearchResults1"
                    :key="person.id"
                    @click="selectPerson1(person)"
                    class="p-2 hover:bg-muted cursor-pointer text-sm"
                  >
                    <div class="font-medium">
                      {{ getGenealogyName(person) }}
                    </div>
                    <div class="text-xs text-muted-foreground">
                      {{ person.gender === 1 ? "男" : "女" }} · 世系:
                      {{ person.lineage }}
                    </div>
                  </div>
                </div>
              </div>
              <Button
                size="sm"
                @click="handleSearchPerson1"
                :disabled="!ancestorSearchValue1.trim()"
              >
                搜索
              </Button>
              <Button
                size="sm"
                variant="outline"
                @click="clearPerson1"
                v-if="selectedPerson1"
              >
                清空
              </Button>
            </div>
            <div v-if="selectedPerson1" class="text-sm text-green-600">
              ✓ 已选择: {{ getGenealogyName(selectedPerson1) }}
            </div>
          </div>

          <!-- 第二个人搜索 -->
          <div class="space-y-2">
            <label class="text-sm font-medium">第二个人</label>
            <div class="flex space-x-2">
              <div class="relative flex-1">
                <Input
                  v-model="ancestorSearchValue2"
                  placeholder="输入姓名搜索..."
                  @input="onInput2Change"
                  @keyup.enter="handleSearchPerson2"
                  :class="selectedPerson2 ? 'border-green-500' : ''"
                />
                <div
                  v-if="ancestorSearchResults2.length > 0"
                  class="absolute top-full left-0 right-0 bg-card border rounded-md shadow-lg z-10 max-h-40 overflow-y-auto"
                >
                  <div class="px-2 py-1 text-xs text-muted-foreground border-b">
                    找到 {{ ancestorSearchResults2.length }} 个结果，点击选择
                  </div>
                  <div
                    v-for="person in ancestorSearchResults2"
                    :key="person.id"
                    @click="selectPerson2(person)"
                    class="p-2 hover:bg-muted cursor-pointer text-sm"
                  >
                    <div class="font-medium">
                      {{ getGenealogyName(person) }}
                    </div>
                    <div class="text-xs text-muted-foreground">
                      {{ person.gender === 1 ? "男" : "女" }} · 世系:
                      {{ person.lineage }}
                    </div>
                  </div>
                </div>
              </div>
              <Button
                size="sm"
                @click="handleSearchPerson2"
                :disabled="!ancestorSearchValue2.trim()"
              >
                搜索
              </Button>
              <Button
                size="sm"
                variant="outline"
                @click="clearPerson2"
                v-if="selectedPerson2"
              >
                清空
              </Button>
            </div>
            <div v-if="selectedPerson2" class="text-sm text-green-600">
              ✓ 已选择: {{ getGenealogyName(selectedPerson2) }}
            </div>
          </div>

          <!-- 操作按钮 -->
          <div class="flex space-x-2 pt-4">
            <Button
              @click="findAncestor"
              :disabled="!selectedPerson1 || !selectedPerson2"
              class="flex-1"
            >
              查找共同先祖
            </Button>
            <Button variant="outline" @click="resetAncestorTool" class="flex-1">
              重置
            </Button>
          </div>
        </div>
      </div>
    </div>

    <!-- 共同先祖结果弹窗 -->
    <div
      v-if="showAncestorResult && ancestorResult"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-card rounded-lg shadow-xl max-w-md w-full">
        <div class="flex items-center justify-between p-4 border-b">
          <h2 class="text-lg font-semibold">查找结果</h2>
          <Button variant="ghost" size="sm" @click="showAncestorResult = false">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-4 space-y-4">
          <div class="text-center">
            <div class="text-xl font-bold text-primary">
              {{ getTreeNodeLabel(ancestorResult) }}
            </div>
            <div class="flex justify-center space-x-2 mt-2">
              <span
                class="inline-flex items-center px-2 py-1 rounded-full text-xs bg-muted"
              >
                {{ ancestorResult.gender === 1 ? "男" : "女" }}
              </span>
              <span
                class="inline-flex items-center px-2 py-1 rounded-full text-xs bg-muted"
              >
                世系: {{ ancestorResult.lineage }}
              </span>
            </div>
            <div class="mt-2 text-sm text-muted-foreground">
              {{ ancestorResult.desc }}
            </div>
          </div>

          <div class="flex space-x-2">
            <Button @click="locateAncestor" class="flex-1"> 定位到节点 </Button>
            <Button
              variant="outline"
              @click="showAncestorResult = false"
              class="flex-1"
            >
              关闭
            </Button>
          </div>
        </div>
      </div>
    </div>

    <!-- 祖宗十八代搜索弹窗 -->
    <div
      v-if="showEighteenModal"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div
        class="bg-card rounded-lg shadow-xl max-w-md w-full max-h-[90vh] overflow-y-auto"
      >
        <div class="flex items-center justify-between p-4 border-b">
          <h2 class="text-lg font-semibold">祖宗十八代</h2>
          <Button variant="ghost" size="sm" @click="resetEighteenTool">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-4 space-y-4">
          <div class="text-sm text-muted-foreground mb-4">
            请输入姓名搜索，然后从下拉列表中选择准确的人物
          </div>

          <!-- 人员搜索 -->
          <div class="space-y-2">
            <label class="text-sm font-medium">选择人物</label>
            <div class="flex space-x-2">
              <div class="relative flex-1">
                <Input
                  v-model="eighteenSearchValue"
                  placeholder="输入姓名搜索..."
                  @input="onInputEighteenChange"
                  @keyup.enter="handleSearchPersonEighteen"
                  :class="selectedPersonEighteen ? 'border-green-500' : ''"
                />
                <div
                  v-if="eighteenSearchResults.length > 0"
                  class="absolute top-full left-0 right-0 bg-card border rounded-md shadow-lg z-10 max-h-40 overflow-y-auto"
                >
                  <div class="px-2 py-1 text-xs text-muted-foreground border-b">
                    找到 {{ eighteenSearchResults.length }} 个结果，点击选择
                  </div>
                  <div
                    v-for="person in eighteenSearchResults"
                    :key="person.id"
                    @click="selectPersonEighteen(person)"
                    class="p-2 hover:bg-muted cursor-pointer text-sm"
                  >
                    <div class="font-medium">
                      {{ getGenealogyName(person) }}
                    </div>
                    <div class="text-xs text-muted-foreground">
                      {{ person.gender === 1 ? "男" : "女" }} · 世系:
                      {{ person.lineage }}
                    </div>
                  </div>
                </div>
              </div>
              <Button
                size="sm"
                @click="handleSearchPersonEighteen"
                :disabled="!eighteenSearchValue.trim()"
              >
                搜索
              </Button>
              <Button
                size="sm"
                variant="outline"
                @click="clearPersonEighteen"
                v-if="selectedPersonEighteen"
              >
                清空
              </Button>
            </div>
            <div v-if="selectedPersonEighteen" class="text-sm text-green-600">
              ✓ 已选择: {{ getGenealogyName(selectedPersonEighteen) }}
            </div>
          </div>

          <!-- 操作按钮 -->
          <div class="flex space-x-2 pt-4">
            <Button
              @click="generateEighteen"
              :disabled="!selectedPersonEighteen"
              class="flex-1"
            >
              生成祖宗十八代
            </Button>
            <Button variant="outline" @click="resetEighteenTool" class="flex-1">
              重置
            </Button>
          </div>
        </div>
      </div>
    </div>

    <!-- 祖宗十八代结果弹窗 -->
    <div
      v-if="showEighteenResultModal"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div
        class="bg-card rounded-lg shadow-xl max-w-lg w-full max-h-[90vh] overflow-y-auto"
      >
        <div class="flex items-center justify-between p-4 border-b">
          <h2 class="text-lg font-semibold">祖宗十八代</h2>
          <Button
            variant="ghost"
            size="sm"
            @click="showEighteenResultModal = false"
          >
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-4">
          <div class="space-y-3 max-h-[70vh] overflow-y-auto">
            <!-- 上九代 -->
            <div
              v-for="(before, index) in eighteenData.beforeList"
              :key="'before-' + index"
              class="flex items-center space-x-3 p-2 border rounded-lg hover:bg-muted/50"
            >
              <div class="flex-shrink-0 w-16">
                <span
                  class="inline-flex items-center px-2 py-1 rounded-full text-xs bg-blue-100 text-blue-800"
                >
                  {{ eighteenData.beforeLabel[index] }}
                </span>
              </div>
              <div class="flex-1" v-if="before">
                <div class="font-medium">{{ getTreeNodeLabel(before) }}</div>
                <div class="text-xs text-muted-foreground space-x-2">
                  <span v-if="before.style_name_level"
                    >字辈: {{ before.style_name_level }}</span
                  >
                  <span>世系: {{ before.lineage }}</span>
                </div>
              </div>
              <div class="flex-1 text-muted-foreground text-sm" v-else>
                无记录
              </div>
            </div>

            <!-- 自己 -->
                         <div
               class="flex items-center space-x-3 p-2 border-2 border-blue-500 rounded-lg bg-blue-50"
             >
              <div class="flex-shrink-0 w-16">
                <span
                  class="inline-flex items-center px-2 py-1 rounded-full text-xs bg-primary text-primary-foreground"
                >
                  自己
                </span>
              </div>
              <div class="flex-1">
                <div class="font-medium text-primary">
                  {{ getTreeNodeLabel(eighteenData.me) }}
                </div>
                <div class="text-xs text-muted-foreground space-x-2">
                  <span v-if="eighteenData.me.style_name_level"
                    >字辈: {{ eighteenData.me.style_name_level }}</span
                  >
                  <span>世系: {{ eighteenData.me.lineage }}</span>
                </div>
              </div>
            </div>

            <!-- 下九代 -->
            <div
              v-for="(after, index) in eighteenData.afterList"
              :key="'after-' + index"
              class="flex items-center space-x-3 p-2 border rounded-lg hover:bg-muted/50"
            >
              <div class="flex-shrink-0 w-16">
                <span
                  class="inline-flex items-center px-2 py-1 rounded-full text-xs bg-green-100 text-green-800"
                >
                  {{ eighteenData.afterLabel[index] }}
                </span>
              </div>
              <div class="flex-1" v-if="after">
                <div class="font-medium">{{ getTreeNodeLabel(after) }}</div>
                <div class="text-xs text-muted-foreground space-x-2">
                  <span v-if="after.style_name_level"
                    >字辈: {{ after.style_name_level }}</span
                  >
                  <span>世系: {{ after.lineage }}</span>
                </div>
              </div>
              <div class="flex-1 text-muted-foreground text-sm" v-else>
                无记录
              </div>
            </div>
          </div>

          <div class="flex space-x-2 pt-4">
            <Button @click="resetEighteenTool" class="flex-1">
              重新查询
            </Button>
            <Button
              variant="outline"
              @click="showEighteenResultModal = false"
              class="flex-1"
            >
              关闭
            </Button>
          </div>
        </div>
      </div>
    </div>

    <!-- 辈分计算搜索弹窗 -->
    <div
      v-if="showGenerationModal"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div
        class="bg-card rounded-lg shadow-xl max-w-md w-full max-h-[90vh] overflow-y-auto"
      >
        <div class="flex items-center justify-between p-4 border-b">
          <h2 class="text-lg font-semibold">辈分计算</h2>
          <Button variant="ghost" size="sm" @click="resetGenerationTool">
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-4 space-y-4">
          <div class="text-sm text-muted-foreground mb-4">
            请输入两个人的姓名进行辈分对比，从下拉列表中选择准确的人物
          </div>

          <!-- 第一个人搜索 -->
          <div class="space-y-2">
            <label class="text-sm font-medium">第一个人</label>
            <div class="flex space-x-2">
              <div class="relative flex-1">
                <Input
                  v-model="generationSearchValue1"
                  placeholder="输入姓名搜索..."
                  @input="onInputGenerationChange1"
                  @keyup.enter="handleSearchGenerationPerson1"
                  :class="selectedGenerationPerson1 ? 'border-green-500' : ''"
                />
                <div
                  v-if="generationSearchResults1.length > 0"
                  class="absolute top-full left-0 right-0 bg-card border rounded-md shadow-lg z-10 max-h-40 overflow-y-auto"
                >
                  <div class="px-2 py-1 text-xs text-muted-foreground border-b">
                    找到 {{ generationSearchResults1.length }} 个结果，点击选择
                  </div>
                  <div
                    v-for="person in generationSearchResults1"
                    :key="person.id"
                    @click="selectGenerationPerson1(person)"
                    class="p-2 hover:bg-muted cursor-pointer text-sm"
                  >
                    <div class="font-medium">
                      {{ getGenealogyName(person) }}
                    </div>
                    <div class="text-xs text-muted-foreground">
                      {{ person.gender === 1 ? "男" : "女" }} · 字辈:
                      {{ person.style_name_level || "无" }} · 世系:
                      {{ person.lineage }}
                    </div>
                  </div>
                </div>
              </div>
              <Button
                size="sm"
                @click="handleSearchGenerationPerson1"
                :disabled="!generationSearchValue1.trim()"
              >
                搜索
              </Button>
              <Button
                size="sm"
                variant="outline"
                @click="clearGenerationPerson1"
                v-if="selectedGenerationPerson1"
              >
                清空
              </Button>
            </div>
            <div
              v-if="selectedGenerationPerson1"
              class="text-sm text-green-600"
            >
              ✓ 已选择: {{ getGenealogyName(selectedGenerationPerson1) }}
            </div>
          </div>

          <!-- 第二个人搜索 -->
          <div class="space-y-2">
            <label class="text-sm font-medium">第二个人</label>
            <div class="flex space-x-2">
              <div class="relative flex-1">
                <Input
                  v-model="generationSearchValue2"
                  placeholder="输入姓名搜索..."
                  @input="onInputGenerationChange2"
                  @keyup.enter="handleSearchGenerationPerson2"
                  :class="selectedGenerationPerson2 ? 'border-green-500' : ''"
                />
                <div
                  v-if="generationSearchResults2.length > 0"
                  class="absolute top-full left-0 right-0 bg-card border rounded-md shadow-lg z-10 max-h-40 overflow-y-auto"
                >
                  <div class="px-2 py-1 text-xs text-muted-foreground border-b">
                    找到 {{ generationSearchResults2.length }} 个结果，点击选择
                  </div>
                  <div
                    v-for="person in generationSearchResults2"
                    :key="person.id"
                    @click="selectGenerationPerson2(person)"
                    class="p-2 hover:bg-muted cursor-pointer text-sm"
                  >
                    <div class="font-medium">
                      {{ getGenealogyName(person) }}
                    </div>
                    <div class="text-xs text-muted-foreground">
                      {{ person.gender === 1 ? "男" : "女" }} · 字辈:
                      {{ person.style_name_level || "无" }} · 世系:
                      {{ person.lineage }}
                    </div>
                  </div>
                </div>
              </div>
              <Button
                size="sm"
                @click="handleSearchGenerationPerson2"
                :disabled="!generationSearchValue2.trim()"
              >
                搜索
              </Button>
              <Button
                size="sm"
                variant="outline"
                @click="clearGenerationPerson2"
                v-if="selectedGenerationPerson2"
              >
                清空
              </Button>
            </div>
            <div
              v-if="selectedGenerationPerson2"
              class="text-sm text-green-600"
            >
              ✓ 已选择: {{ getGenealogyName(selectedGenerationPerson2) }}
            </div>
          </div>

          <!-- 操作按钮 -->
          <div class="flex space-x-2 pt-4">
            <Button
              @click="calculateGeneration"
              :disabled="
                !selectedGenerationPerson1 || !selectedGenerationPerson2
              "
              class="flex-1"
            >
              计算辈分
            </Button>
            <Button
              variant="outline"
              @click="resetGenerationTool"
              class="flex-1"
            >
              重置
            </Button>
          </div>
        </div>
      </div>
    </div>

    <!-- 辈分计算结果弹窗 -->
    <div
      v-if="showGenerationResultModal && generationResult"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-card rounded-lg shadow-xl max-w-lg w-full">
        <div class="flex items-center justify-between p-4 border-b">
          <h2 class="text-lg font-semibold">辈分计算结果</h2>
          <Button
            variant="ghost"
            size="sm"
            @click="showGenerationResultModal = false"
          >
            <svg
              class="w-4 h-4"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </Button>
        </div>

        <div class="p-6 space-y-6">
          <!-- 对比结果 -->
                     <div
             class="text-center p-4 bg-blue-50 rounded-lg border border-blue-200"
           >
            <h3 class="text-lg font-semibold text-primary mb-2">辈分关系</h3>
            <p class="text-xl font-bold">{{ generationResult.relationship }}</p>
          </div>

          <!-- 详细信息对比 -->
          <div class="grid grid-cols-2 gap-4">
            <!-- 第一个人 -->
            <div class="space-y-3 p-4 border rounded-lg">
              <div class="text-sm font-medium text-muted-foreground">
                第一个人
              </div>
              <div class="space-y-2">
                <div class="font-medium text-lg">
                  {{ getTreeNodeLabel(generationResult.person1) }}
                </div>
                <div class="space-y-1 text-sm">
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">性别:</span>
                    <span
                      :class="
                        generationResult.person1.gender === 1
                          ? 'text-blue-600'
                          : 'text-pink-600'
                      "
                    >
                      {{ generationResult.person1.gender === 1 ? "男" : "女" }}
                    </span>
                  </div>
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">字辈:</span>
                    <span class="font-medium text-primary">{{
                      generationResult.person1.style_name_level || "无"
                    }}</span>
                  </div>
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">世系:</span>
                    <span class="font-medium text-primary">{{
                      generationResult.person1.lineage
                    }}</span>
                  </div>
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">房派:</span>
                    <span>{{ generationResult.person1.faction || "无" }}</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- 第二个人 -->
            <div class="space-y-3 p-4 border rounded-lg">
              <div class="text-sm font-medium text-muted-foreground">
                第二个人
              </div>
              <div class="space-y-2">
                <div class="font-medium text-lg">
                  {{ getTreeNodeLabel(generationResult.person2) }}
                </div>
                <div class="space-y-1 text-sm">
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">性别:</span>
                    <span
                      :class="
                        generationResult.person2.gender === 1
                          ? 'text-blue-600'
                          : 'text-pink-600'
                      "
                    >
                      {{ generationResult.person2.gender === 1 ? "男" : "女" }}
                    </span>
                  </div>
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">字辈:</span>
                    <span class="font-medium text-primary">{{
                      generationResult.person2.style_name_level || "无"
                    }}</span>
                  </div>
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">世系:</span>
                    <span class="font-medium text-primary">{{
                      generationResult.person2.lineage
                    }}</span>
                  </div>
                  <div class="flex justify-between">
                    <span class="text-muted-foreground">房派:</span>
                    <span>{{ generationResult.person2.faction || "无" }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- 统计信息 -->
                     <div class="p-4 bg-gray-50 rounded-lg">
            <div class="text-sm space-y-2">
              <div class="flex justify-between">
                <span class="text-muted-foreground">世系差值:</span>
                <span class="font-medium">
                  {{ Math.abs(generationResult.lineageDiff) }}
                  {{ generationResult.lineageDiff === 0 ? "(同辈)" : "辈" }}
                </span>
              </div>
              <div
                class="flex justify-between"
                v-if="
                  generationResult.person1.faction &&
                  generationResult.person2.faction
                "
              >
                <span class="text-muted-foreground">房派关系:</span>
                <span class="font-medium">
                  {{
                    generationResult.person1.faction ===
                    generationResult.person2.faction
                      ? "同房派"
                      : "不同房派"
                  }}
                </span>
              </div>
            </div>
          </div>

          <div class="flex space-x-2">
            <Button @click="resetGenerationTool" class="flex-1">
              重新计算
            </Button>
            <Button
              variant="outline"
              @click="showGenerationResultModal = false"
              class="flex-1"
            >
              关闭
            </Button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, onUnmounted } from "vue";
import { Button } from "./ui/button";
import { Input } from "./ui/input";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "./ui/dropdown-menu";
import G6 from "@antv/g6";

// 响应式数据
const searchValue = ref("");
const loading = ref(true);
const error = ref("");
const genealogyData = ref(null);
const graphContainer = ref<HTMLDivElement>();
const showNodeModal = ref(false);
const showSearchResult = ref(false);
const modalData = ref<any>({});
const searchResult = ref<any[]>([]);

// 共同先祖功能相关
const showAncestorModal = ref(false);
const ancestorSearchValue1 = ref("");
const ancestorSearchValue2 = ref("");
const ancestorSearchResults1 = ref<any[]>([]);
const ancestorSearchResults2 = ref<any[]>([]);
const selectedPerson1 = ref<any>(null);
const selectedPerson2 = ref<any>(null);
const ancestorResult = ref<any>(null);
const showAncestorResult = ref(false);

// 祖宗十八代功能相关
const showEighteenModal = ref(false);
const eighteenSearchValue = ref("");
const eighteenSearchResults = ref<any[]>([]);
const selectedPersonEighteen = ref<any>(null);

// 定义祖宗十八代数据类型
interface EighteenData {
  beforeList: any[];
  me: any;
  afterList: any[];
  beforeLabel: string[];
  afterLabel: string[];
}

const eighteenData = ref<EighteenData>({
  beforeList: [],
  me: {},
  afterList: [],
  beforeLabel: [
    "鼻祖",
    "远祖",
    "太祖",
    "烈祖",
    "天祖",
    "高祖",
    "曾祖",
    "祖父",
    "父亲",
  ],
  afterLabel: [
    "儿子",
    "孙子",
    "曾孙",
    "玄孙",
    "来孙",
    "晜孙",
    "仍孙",
    "云孙",
    "耳孙",
  ],
});
const showEighteenResultModal = ref(false);

// 辈分计算功能相关
const showGenerationModal = ref(false);
const generationSearchValue1 = ref("");
const generationSearchValue2 = ref("");
const generationSearchResults1 = ref<any[]>([]);
const generationSearchResults2 = ref<any[]>([]);
const selectedGenerationPerson1 = ref<any>(null);
const selectedGenerationPerson2 = ref<any>(null);
const generationResult = ref<any>(null);
const showGenerationResultModal = ref(false);

// G6 相关
let graph: any = null;
const treeData = ref<any>({});

// 节点基础配置
const nodeBaseConfig = {
  width: 160,
  height: 50,
  borderWidth: 1.5,
  borderRadius: 6,
  labelFontSize: 14,
  descriptionFontSize: 12,
};

// 获取节点配置（根据性别）
const getNodeConfig = (node: any) => {
  if (node.gender === 1) {
    // 男性节点
    return {
      ...nodeBaseConfig,
      bgColor: "#ECECE2",
      borderColor: "#EC826D",
      descriptionColor: "#EC9D03",
      labelFontColor: "#00386C",
    };
  } else {
    // 女性节点
    return {
      ...nodeBaseConfig,
      bgColor: "#ffb3a7",
      borderColor: "#EC826D",
      descriptionColor: "#9c5333",
      labelFontColor: "#00386C",
    };
  }
};

// 获取树节点标签
const getTreeNodeLabel = (node: any) => {
  if (!node) return "";
  if (node.posthumous_title) {
    return node.posthumous_title;
  } else if (node.style_name && node.name) {
    return `${node.name} 字:${node.style_name}`;
  } else {
    return node.name || node.style_name || "未知";
  }
};

// 获取族谱姓名格式
const getGenealogyName = (node: any) => {
  if (!node) return "";
  const parts = [];
  if (node.posthumous_title) parts.push(node.posthumous_title);
  if (node.style_name) parts.push(`字:${node.style_name}`);
  if (node.name) parts.push(`名:${node.name}`);
  return parts.filter(Boolean).join("｜") || "未知";
};

// 获取节点描述
const getDescription = (node: any) => {
  if (node.spouses && node.spouses.length > 0) {
    return node.spouses
      .map((item: any) => `${item.role} ${item.spouse.name}`)
      .join("｜");
  }
  return node.status || "";
};

// 格式化生卒年
const formatBirthDeathDate = (dateObj: any) => {
  if (!dateObj) return null;

  const parts = [];
  if (dateObj.year) parts.push(`${dateObj.year}年`);
  if (dateObj.month) parts.push(dateObj.month);
  if (dateObj.day) parts.push(dateObj.day);
  if (dateObj.hour) parts.push(dateObj.hour);
  if (dateObj.ganzhi) parts.push(`(${dateObj.ganzhi})`);

  return parts.length > 0 ? parts.join(" ") : null;
};

// 创建G6图谱
const createGraph = (data: any) => {
  // 如果已存在图实例，先销毁
  if (graph) {
    graph.destroy();
    graph = null;
  }

  // 注册自定义节点
  G6.registerNode(
    "genealogy-card",
    {
      draw(cfg: any, group: any) {
        const config = getNodeConfig(cfg);
        // const width = config.width;
        // const height = config.height;

        // 创建节点容器
        const container = group.addShape("rect", {
          attrs: {
            x: 0,
            y: 0,
            width: 160,
            height: 50,
            fill: config.bgColor,
            stroke: config.borderColor,
            lineWidth: config.borderWidth,
            radius: config.borderRadius,
            cursor: "pointer",
          },
          name: "big-rect-shape",
        });

        // 添加标签文字
        const labelShape = group.addShape("text", {
          attrs: {
            text: cfg.label,
            x: 8,
            y: cfg.description ? 19.5 : 25,
            fontSize: config.labelFontSize,
            textAlign: "start",
            textBaseline: "bottom",
            fill: config.labelFontColor,
            cursor: "pointer",
          },
          name: "label-text-shape",
        });

        // 添加描述文字
        if (cfg.description) {
          group.addShape("text", {
            attrs: {
              text: cfg.description,
              x: 8,
              y: 50 - 6.5,
              fontSize: config.descriptionFontSize,
              textAlign: "start",
              textBaseline: "bottom",
              fill: config.descriptionColor,
            },
            name: "description-text-shape",
          });
        }

        // 点击事件 - 支持移动端触摸
        const clickHandler = (e: any) => {
          e.preventDefault();
          e.stopPropagation();
          console.log("节点被点击:", cfg.id, cfg.name);
          handleClickNode(cfg);
        };

        container.on("click", clickHandler);
        container.on("touchstart", clickHandler);

        labelShape.on("click", clickHandler);
        labelShape.on("touchstart", clickHandler);

        // 展开/收起图标
        const hasChildren = cfg.children && cfg.children.length > 0;
        if (hasChildren) {
          const expandIcon = group.addShape("marker", {
            attrs: {
              x: 175,
              y: 12,
              r: 10,
              symbol: cfg.collapsed ? G6.Marker.expand : G6.Marker.collapse,
              stroke: "#666",
              lineWidth: 1.5,
            },
            name: "collapse-icon",
          });

          expandIcon.on("click", () => {
            cfg.collapsed = !cfg.collapsed;
            const icon = group.find(
              (e: any) => e.get("name") === "collapse-icon"
            );
            icon.attr(
              "symbol",
              cfg.collapsed ? G6.Marker.expand : G6.Marker.collapse
            );
            graph.layout();
          });
        }

        return container;
      },
      update(cfg: any, node: any) {
        const group = node.getContainer();
        const icon = group.find((e: any) => e.get("name") === "collapse-icon");
        if (icon) {
          icon.attr(
            "symbol",
            cfg.collapsed ? G6.Marker.expand : G6.Marker.collapse
          );
        }
      },
    },
    "modelRect"
  );

  // 数据预处理
  G6.Util.traverseTree(data, (sub: any) => {
    sub.id = sub.id.toString();
    sub.collapsed = false;
  });

  // 获取容器尺寸
  const containerElement = document.getElementById("mountNode");
  if (!containerElement) {
    console.error("容器未找到");
    return;
  }

  const rect = containerElement.getBoundingClientRect();
  let width = rect.width || containerElement.clientWidth;
  let height = rect.height || containerElement.clientHeight;

  // 如果容器尺寸仍然为0，使用默认尺寸
  if (width === 0 || height === 0) {
    console.warn("容器尺寸为0，使用默认尺寸");
    const parentElement = containerElement.parentElement;
    if (parentElement) {
      const parentRect = parentElement.getBoundingClientRect();
      width = parentRect.width || 800;
      height = parentRect.height || 600;
    } else {
      width = 800;
      height = 600;
    }
  }

  console.log("最终使用尺寸:", { width, height });

  try {
    // 创建图实例 - 使用实际容器尺寸
    graph = new G6.TreeGraph({
      container: "mountNode",
      width: width,
      height: height,
      fitView: true,
      modes: {
        default: ["drag-canvas", "zoom-canvas"],
      },
      defaultNode: {
        type: "genealogy-card",
      },
      defaultEdge: {
        type: "polyline",
        style: {
          stroke: "#0A0A0A",
          endArrow: true,
          startArrow: false,
          offset: 30,
        },
      },
      layout: {
        type: "mindmap",
        direction: "V",
        getId: function getId(d: any) {
          return d.id;
        },
        getHeight: function getHeight() {
          return 14;
        },
        getWidth: function getWidth() {
          return 80;
        },
        getVGap: function getVGap() {
          return 30;
        },
        getHGap: function getHGap() {
          return 80;
        },
      },
    });

    // 配置节点
    graph.node(function (node: any) {
      return {
        label: getTreeNodeLabel(node),
        description: getDescription(node),
        labelCfg: {
          offset: 10,
          position:
            node.children && node.children.length > 0 ? "left" : "right",
        },
      };
    });

    graph.data(data);
    graph.render();
    graph.fitView();

    // 添加节点点击事件监听（确保移动端兼容性）
    graph.on("node:click", (e: any) => {
      console.log("G6节点点击事件:", e.item.getModel());
      handleClickNode(e.item.getModel());
    });

    graph.on("node:touchstart", (e: any) => {
      console.log("G6节点触摸事件:", e.item.getModel());
      handleClickNode(e.item.getModel());
    });

    console.log("G6图谱创建成功");
  } catch (err) {
    console.error("G6图谱创建失败:", err);
    error.value = "图谱渲染失败，请刷新页面重试";
  }
};

// 数据获取函数
const fetchData = async () => {
  try {
    loading.value = true;
    error.value = "";

    const response = await fetch(
      "https://genealogy.ameno.space/api/data"
    );

    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }

    const result = await response.json();

    if (result.data && Array.isArray(result.data) && result.data.length > 0) {
      genealogyData.value = result.data[0];
      treeData.value = result.data[0];
      console.log("族谱数据加载成功:", genealogyData.value);

      // 等待DOM更新和容器挂载
      await nextTick();

      // 延迟一下确保容器完全渲染和尺寸计算完成
      setTimeout(() => {
        initializeGraph(result.data[0]);
      }, 500);
    } else {
      throw new Error("数据格式错误或数据为空");
    }
  } catch (err) {
    error.value = err instanceof Error ? err.message : "未知错误";
    console.error("获取族谱数据失败:", err);
    loading.value = false;
  }
};

// 初始化图谱（带重试机制）
const initializeGraph = (data: any, retryCount = 0) => {
  const maxRetries = 3;

  try {
    const containerElement = document.getElementById("mountNode");

    if (!containerElement) {
      if (retryCount < maxRetries) {
        console.log(`容器未找到，重试第 ${retryCount + 1} 次...`);
        setTimeout(() => initializeGraph(data, retryCount + 1), 200);
        return;
      } else {
        throw new Error("无法找到图谱容器");
      }
    }

    // 检查容器是否可见
    const rect = containerElement.getBoundingClientRect();
    const isVisible = containerElement.offsetParent !== null;

    if (!isVisible || (rect.width === 0 && rect.height === 0)) {
      if (retryCount < maxRetries) {
        console.log(`容器不可见或尺寸无效，重试第 ${retryCount + 1} 次...`, {
          rect,
          isVisible,
        });
        setTimeout(() => initializeGraph(data, retryCount + 1), 300);
        return;
      } else {
        console.warn("容器状态异常，强制创建图谱");
      }
    }

    // 无论如何都尝试创建图谱
    createGraph(data);
    loading.value = false;
    console.log("图谱初始化成功");
  } catch (err) {
    console.error("图谱初始化失败:", err);
    error.value = err instanceof Error ? err.message : "图谱初始化失败";
    loading.value = false;
  }
};

// 搜索处理
const handleSearchGenealogy = () => {
  if (!searchValue.value.trim()) return;

  // 简单的搜索实现
  const results: any[] = [];
  const searchInNode = (node: any) => {
    if (
      node.name?.includes(searchValue.value) ||
      node.style_name?.includes(searchValue.value) ||
      node.posthumous_title?.includes(searchValue.value)
    ) {
      results.push(node);
    }
    if (node.children) {
      node.children.forEach(searchInNode);
    }
  };

  if (treeData.value) {
    searchInNode(treeData.value);
  }

  searchResult.value = results.slice(0, 20); // 限制结果数量
  showSearchResult.value = true;
};

// 点击搜索结果
const handleSearchResultClick = (id: string) => {
  if (graph) {
    graph.zoomTo(1.4);
    graph.focusItem(id, true);
    showSearchResult.value = false;
  }
};

// 点击节点
const handleClickNode = (data: any) => {
  console.log("处理节点点击事件:", data);
  modalData.value = data;
  showNodeModal.value = true;
};

// 搜索人员API
const searchPerson = async (query: string): Promise<any[]> => {
  if (!query.trim()) return [];

  try {
    const response = await fetch(
      `https://genealogy.ameno.space/api/genealogy/search?q=${encodeURIComponent(
        query
      )}`
    );

    if (!response.ok) {
      throw new Error(`搜索失败: ${response.status}`);
    }

    const result = await response.json();
    return result.success ? result.data || [] : [];
      } catch (error: any) {
      console.error("搜索人员失败:", error);
      return [];
    }
};

// 查找共同先祖
const findCommonAncestor = (person1Id: string, person2Id: string): any => {
  if (!treeData.value) return null;

  // 获取从根节点到目标节点的路径
  const getPathToNode = (
    targetId: string,
    node: any,
    path: any[] = []
  ): any[] | null => {
    path.push(node);

    if (node.id === targetId) {
      return [...path];
    }

    if (node.children) {
      for (const child of node.children) {
        const result = getPathToNode(targetId, child, path);
        if (result) return result;
      }
    }

    path.pop();
    return null;
  };

  const path1 = getPathToNode(person1Id, treeData.value);
  const path2 = getPathToNode(person2Id, treeData.value);

  if (!path1 || !path2) return null;

  // 找到最近的共同祖先
  let commonAncestor = null;
  const minLength = Math.min(path1.length, path2.length);

  for (let i = 0; i < minLength; i++) {
    if (path1[i].id === path2[i].id) {
      commonAncestor = path1[i];
    } else {
      break;
    }
  }

  return commonAncestor;
};

// 根据ID查找祖先（向上追溯）
const findAncestorsById = (
  tree: any,
  targetId: string,
  levels: number = 9
): any[] => {
  const ancestors: any[] = [];

  const findAncestors = (node: any, currentLevel: number, parent?: any) => {
    if (parent) {
      node.parent = parent; // 记录父级节点
    }

    if (node.id === targetId) {
      const { children, ...nodeWithoutChildren } = node;
      ancestors.unshift(nodeWithoutChildren);

      let currentNode = node;
      for (let i = 1; i < Math.min(currentLevel, levels); i++) {
        currentNode = currentNode.parent; // 获取父级节点
        if (!currentNode) break; // 如果已经没有父级节点了，提前结束
        const { children, ...ancestorWithoutChildren } = currentNode;
        ancestors.unshift(ancestorWithoutChildren);
      }
      return true;
    }

    if (node.children && node.children.length > 0) {
      for (const child of node.children) {
        if (findAncestors(child, currentLevel + 1, node)) {
          return true;
        }
      }
    }

    return false;
  };

  findAncestors(tree, 0);

  // 补充 null 到数组开头，确保数组长度为指定值
  const nullCount = Math.max(0, levels - ancestors.length);
  for (let i = 0; i < nullCount; i++) {
    ancestors.unshift(null);
  }

  return ancestors;
};

// 查找后代（向下追溯）
const findNodesByIdAndSortAndGender = (
  node: any,
  targetId: string,
  currentDepth: number
): any[] | null => {
  if (node.id === targetId) {
    return getNodeHierarchy(node, currentDepth);
  }

  if (node.children) {
    for (const child of node.children) {
      const result = findNodesByIdAndSortAndGender(
        child,
        targetId,
        currentDepth + 1
      );
      if (result) {
        return result;
      }
    }
  }

  return null;
};

const getNodeHierarchy = (node: any, depth: number): any[] => {
  if (depth === 8) {
    return [];
  }

  const { children, ...rest } = node; // 保存原始对象中除了 children 属性之外的属性
  const nextDepth = depth + 1;

  const matchingChildren = (children || [])
    .filter(
      (child: any) =>
        child.sort === 1 &&
        child.gender === 1 &&
        child.status !== "失考" &&
        child.status !== "失传" &&
        child.status !== "不详"
    )
    .map((child: any) => getNodeHierarchy(child, nextDepth));

  while (matchingChildren.length < 8 - depth) {
    matchingChildren.push(null);
  }

  return [{ ...rest }, ...matchingChildren.flat()]; // 返回包含原始对象（除了 children）的数组
};

// 工具选择处理
const handleSelectTool = (value: string) => {
  console.log("工具选择:", value);

  switch (value) {
    case "ancestor":
      // 打开共同先祖搜索弹窗
      showAncestorModal.value = true;
      break;
    case "level":
      // 打开辈分计算搜索弹窗
      showGenerationModal.value = true;
      break;
    case "eighteen":
      // 打开祖宗十八代搜索弹窗
      showEighteenModal.value = true;
      break;
    default:
      console.log("未知工具:", value);
  }
};

// 搜索第一个人
const handleSearchPerson1 = async () => {
  if (!ancestorSearchValue1.value.trim()) {
    ancestorSearchResults1.value = [];
    return;
  }

  ancestorSearchResults1.value = await searchPerson(ancestorSearchValue1.value);
};

// 搜索第二个人
const handleSearchPerson2 = async () => {
  if (!ancestorSearchValue2.value.trim()) {
    ancestorSearchResults2.value = [];
    return;
  }

  ancestorSearchResults2.value = await searchPerson(ancestorSearchValue2.value);
};

// 选择第一个人
const selectPerson1 = (person: any) => {
  selectedPerson1.value = person;
  ancestorSearchValue1.value = getGenealogyName(person);
  ancestorSearchResults1.value = [];
};

// 选择第二个人
const selectPerson2 = (person: any) => {
  selectedPerson2.value = person;
  ancestorSearchValue2.value = getGenealogyName(person);
  ancestorSearchResults2.value = [];
};

// 清空第一个人的选择
const clearPerson1 = () => {
  selectedPerson1.value = null;
  ancestorSearchValue1.value = "";
  ancestorSearchResults1.value = [];
};

// 清空第二个人的选择
const clearPerson2 = () => {
  selectedPerson2.value = null;
  ancestorSearchValue2.value = "";
  ancestorSearchResults2.value = [];
};

// 监听输入框变化，如果用户手动修改了已选择的内容，则清空选择
const onInput1Change = () => {
  if (
    selectedPerson1.value &&
    ancestorSearchValue1.value !== getGenealogyName(selectedPerson1.value)
  ) {
    selectedPerson1.value = null;
  }
  ancestorSearchResults1.value = [];
};

const onInput2Change = () => {
  if (
    selectedPerson2.value &&
    ancestorSearchValue2.value !== getGenealogyName(selectedPerson2.value)
  ) {
    selectedPerson2.value = null;
  }
  ancestorSearchResults2.value = [];
};

// 查找共同先祖
const findAncestor = () => {
  if (!selectedPerson1.value || !selectedPerson2.value) {
    alert("请先选择两个人物");
    return;
  }

  const ancestor = findCommonAncestor(
    selectedPerson1.value.id,
    selectedPerson2.value.id
  );

  if (ancestor) {
    ancestorResult.value = ancestor;
    showAncestorResult.value = true;
  } else {
    alert("未找到共同先祖");
  }
};

// 重置共同先祖工具
const resetAncestorTool = () => {
  ancestorSearchValue1.value = "";
  ancestorSearchValue2.value = "";
  ancestorSearchResults1.value = [];
  ancestorSearchResults2.value = [];
  selectedPerson1.value = null;
  selectedPerson2.value = null;
  ancestorResult.value = null;
  showAncestorModal.value = false;
  showAncestorResult.value = false;
};

// 定位到先祖节点
const locateAncestor = () => {
  if (ancestorResult.value && graph) {
    graph.zoomTo(1.4);
    graph.focusItem(ancestorResult.value.id, true);
    resetAncestorTool();
  }
};

// 祖宗十八代搜索
const handleSearchPersonEighteen = async () => {
  if (!eighteenSearchValue.value.trim()) {
    eighteenSearchResults.value = [];
    return;
  }

  eighteenSearchResults.value = await searchPerson(eighteenSearchValue.value);
};

// 选择祖宗十八代的人
const selectPersonEighteen = (person: any) => {
  selectedPersonEighteen.value = person;
  eighteenSearchValue.value = getGenealogyName(person);
  eighteenSearchResults.value = [];
};

// 清空祖宗十八代选择
const clearPersonEighteen = () => {
  selectedPersonEighteen.value = null;
  eighteenSearchValue.value = "";
  eighteenSearchResults.value = [];
};

// 输入框变化监听
const onInputEighteenChange = () => {
  if (
    selectedPersonEighteen.value &&
    eighteenSearchValue.value !== getGenealogyName(selectedPersonEighteen.value)
  ) {
    selectedPersonEighteen.value = null;
  }
  eighteenSearchResults.value = [];
};

// 生成祖宗十八代
const generateEighteen = () => {
  if (!selectedPersonEighteen.value) {
    alert("请先选择人物");
    return;
  }

  if (!treeData.value) {
    alert("数据未加载完成");
    return;
  }

  const targetNode = findAncestorsById(
    treeData.value,
    selectedPersonEighteen.value.id.toString()
  );
  eighteenData.value.beforeList = targetNode.slice(0, 8);
  eighteenData.value.me = targetNode.pop() || selectedPersonEighteen.value;

  const children = findNodesByIdAndSortAndGender(
    treeData.value,
    selectedPersonEighteen.value.id.toString(),
    0
  );
  eighteenData.value.afterList = Array.from(
    { length: 8 },
    (_, index) => (children && children.slice(1)[index]) || null
  );

  showEighteenResultModal.value = true;
};

// 重置祖宗十八代工具
const resetEighteenTool = () => {
  eighteenSearchValue.value = "";
  eighteenSearchResults.value = [];
  selectedPersonEighteen.value = null;
  eighteenData.value = {
    beforeList: [],
    me: {},
    afterList: [],
    beforeLabel: [
      "鼻祖",
      "远祖",
      "太祖",
      "烈祖",
      "天祖",
      "高祖",
      "曾祖",
      "祖父",
      "父亲",
    ],
    afterLabel: [
      "儿子",
      "孙子",
      "曾孙",
      "玄孙",
      "来孙",
      "晜孙",
      "仍孙",
      "云孙",
      "耳孙",
    ],
  };
  showEighteenModal.value = false;
  showEighteenResultModal.value = false;
};

// 辈分计算搜索
const handleSearchGenerationPerson1 = async () => {
  if (!generationSearchValue1.value.trim()) {
    generationSearchResults1.value = [];
    return;
  }

  generationSearchResults1.value = await searchPerson(
    generationSearchValue1.value
  );
};

const handleSearchGenerationPerson2 = async () => {
  if (!generationSearchValue2.value.trim()) {
    generationSearchResults2.value = [];
    return;
  }

  generationSearchResults2.value = await searchPerson(
    generationSearchValue2.value
  );
};

// 选择辈分计算的人
const selectGenerationPerson1 = (person: any) => {
  selectedGenerationPerson1.value = person;
  generationSearchValue1.value = getGenealogyName(person);
  generationSearchResults1.value = [];
};

const selectGenerationPerson2 = (person: any) => {
  selectedGenerationPerson2.value = person;
  generationSearchValue2.value = getGenealogyName(person);
  generationSearchResults2.value = [];
};

// 清空辈分计算选择
const clearGenerationPerson1 = () => {
  selectedGenerationPerson1.value = null;
  generationSearchValue1.value = "";
  generationSearchResults1.value = [];
};

const clearGenerationPerson2 = () => {
  selectedGenerationPerson2.value = null;
  generationSearchValue2.value = "";
  generationSearchResults2.value = [];
};

// 输入框变化监听
const onInputGenerationChange1 = () => {
  if (
    selectedGenerationPerson1.value &&
    generationSearchValue1.value !==
      getGenealogyName(selectedGenerationPerson1.value)
  ) {
    selectedGenerationPerson1.value = null;
  }
  generationSearchResults1.value = [];
};

const onInputGenerationChange2 = () => {
  if (
    selectedGenerationPerson2.value &&
    generationSearchValue2.value !==
      getGenealogyName(selectedGenerationPerson2.value)
  ) {
    selectedGenerationPerson2.value = null;
  }
  generationSearchResults2.value = [];
};

// 计算辈分关系
const calculateGeneration = () => {
  if (!selectedGenerationPerson1.value || !selectedGenerationPerson2.value) {
    alert("请先选择两个人物");
    return;
  }

  const person1 = selectedGenerationPerson1.value;
  const person2 = selectedGenerationPerson2.value;

  // 计算辈分差异（世系值越小，辈分越高）
  const lineageDiff = person1.lineage - person2.lineage;

  let relationship = "";
  if (lineageDiff === 0) {
    if (person1.id === person2.id) {
      relationship = "同一人";
    } else {
      relationship = "同辈关系";
    }
  } else if (lineageDiff < 0) {
    // person1的世系值更小，所以辈分更高（是长辈）
    const absDiff = Math.abs(lineageDiff);
    if (absDiff === 1) {
      relationship = `${getTreeNodeLabel(person1)} 是 ${getTreeNodeLabel(
        person2
      )} 的长辈（相差1辈）`;
    } else {
      relationship = `${getTreeNodeLabel(person1)} 是 ${getTreeNodeLabel(
        person2
      )} 的长辈（相差${absDiff}辈）`;
    }
  } else {
    // person1的世系值更大，所以辈分更低（是晚辈）
    if (lineageDiff === 1) {
      relationship = `${getTreeNodeLabel(person1)} 是 ${getTreeNodeLabel(
        person2
      )} 的晚辈（相差1辈）`;
    } else {
      relationship = `${getTreeNodeLabel(person1)} 是 ${getTreeNodeLabel(
        person2
      )} 的晚辈（相差${lineageDiff}辈）`;
    }
  }

  generationResult.value = {
    person1,
    person2,
    lineageDiff,
    relationship,
  };

  showGenerationResultModal.value = true;
};

// 重置辈分计算工具
const resetGenerationTool = () => {
  generationSearchValue1.value = "";
  generationSearchValue2.value = "";
  generationSearchResults1.value = [];
  generationSearchResults2.value = [];
  selectedGenerationPerson1.value = null;
  selectedGenerationPerson2.value = null;
  generationResult.value = null;
  showGenerationModal.value = false;
  showGenerationResultModal.value = false;
};

// 刷新数据
const refreshData = () => {
  // 销毁现有图实例
  if (graph) {
    graph.destroy();
    graph = null;
  }

  // 重新获取数据
  fetchData();
};

// 图谱操作函数
const resetView = () => {
  if (graph) {
    graph.zoomTo(1);
    graph.fitCenter();
  }
};

const fitView = () => {
  if (graph) {
    graph.fitView();
  }
};

const downloadImage = () => {
  if (graph) {
    graph.downloadFullImage("genealogy", "image/png");
  }
};

// 响应式调整
const handleResize = () => {
  if (graph) {
    const containerElement = document.getElementById("mountNode");
    if (containerElement) {
      const width = containerElement.clientWidth;
      const height = containerElement.clientHeight;
      console.log("窗口调整，新尺寸:", { width, height });

      if (width > 0 && height > 0) {
        graph.changeSize(width, height);
        graph.fitView();
      }
    }
  }
};

// 组件挂载时获取数据
onMounted(async () => {
  await nextTick();
  console.log("组件已挂载，开始获取数据");

  // 确保容器存在并有尺寸
  const checkContainer = () => {
    const container = document.getElementById("mountNode");
    if (container) {
      const rect = container.getBoundingClientRect();
      console.log("容器状态检查:", {
        exists: !!container,
        visible: container.offsetParent !== null,
        width: rect.width,
        height: rect.height,
      });
    }
  };

  checkContainer();
  fetchData();
  window.addEventListener("resize", handleResize);
});

onUnmounted(() => {
  window.removeEventListener("resize", handleResize);
  if (graph) {
    graph.destroy();
  }
});
</script>

<style lang="scss" scoped>
// 移动端优化样式
@media (max-width: 768px) {
  // 确保触摸目标至少 44px
  :deep(.btn) {
    min-height: 44px;
  }

  // 输入框在移动端更大
  :deep(input) {
    font-size: 16px; // 防止iOS缩放
  }
}

// 文本截断样式
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>