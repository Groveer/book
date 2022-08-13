# 保存时获得sudo权限

```shell
:w !sudo tee %
```

1. 命令:`w !{cmd}`，让 `vim` 执行一个外部命令{cmd}，然后把当前缓冲区的内容从 stdin 传入。
2. `tee` 是一个把 `stdin` 保存到文件的小工具。
3. 而 `%`，是`vim`当中一个只读寄存器的名字，总保存着当前编辑文件的文件路径。

# 简单高效的vim配置

创建`~/.vimrc`文件并输入：
```vim
set nocompatible    " 关闭兼容模式
syntax enable       " 语法高亮
" 开启文件类型侦测
filetype on
" 根据侦测到的不同类型加载对应的插件
filetype plugin on  " 文件类型插件
filetype indent on
set autoindent
autocmd BufEnter * :syntax sync fromstart
" 设置背景透明
hi Normal ctermfg=252 ctermbg=none

set nu              " 显示行号
set showcmd         " 显示命令
set lz              " 当运行宏时，在命令执行完成之前，不重绘屏幕
set hid             " 可以在没有保存的情况下切换buffer
set backspace=eol,start,indent
set whichwrap+=<,>,h,l " 退格键和方向键可以换行
set incsearch       " 增量式搜索
set hlsearch        " 搜索时，高亮显示匹配结果。
" set ignorecase      " 搜索时忽略大小写
set magic           " 额，自己:h magic吧，一行很难解释
set showmatch       " 光标遇到圆括号、方括号、大括号时，自动高亮对应的另一个圆括号、方括号和大括号。
set nowb
set nobackup
set noswapfile      " 不使用swp文件，注意，错误退出后无法恢复
set lbr             " 在breakat字符处而不是最后一个字符处断行
set ai              " 自动缩进
set si              " 智能缩进
set cindent         " C/C++风格缩进

"命令模式下，底部操作指令按下 Tab 键自动补全。第一次按下 Tab，会显示所有匹配的操作指令的清单；第二次按下 Tab，会依次选择各个指令。
set wildmenu
set wildmode=longest:list,full

set nofen
set fdl=10
" tab转化为4个字符
set expandtab
set smarttab
set shiftwidth=4
set tabstop=4
" 不使用beep或flash
set vb t_vb=
"colorscheme elflord
" 启用256色
set t_Co=256
set t_ut=n

set history=50  " vim记住的历史操作的数量，默认的是20
set autoread     " 当文件在外部被修改时，自动重新读取
set mouse=a      " 在所有模式下都允许使用鼠标，还可以是n,v,i,c等

" 光标所在的当前行高亮
set cursorline
" highlight CursorLine   cterm=NONE ctermbg=gray ctermfg=black guibg=NONE guifg=NONE
" highlight CocHighlightText cterm=NONE ctermbg=yellow ctermfg=black guibg=NONE guifg=NONE


" 插件窗口的宽度，如TagList,NERD_tree等，自己设置
let s:PlugWinSize = 28

" 自动完成括号和引号
inoremap <leader>1 ()<esc>:let leavechar=")"<cr>i
inoremap <leader>2 []<esc>:let leavechar="]"<cr>i
inoremap <leader>3 {}<esc>:let leavechar="}"<cr>i
inoremap <leader>4 {<esc>o}<esc>:let leavechar="}"<cr>O
inoremap <leader>q ''<esc>:let leavechar="'"<cr>i
inoremap <leader>w ""<esc>:let leavechar='"'<cr>i

" 用 */# 向 前/后 搜索光标下的单词
vnoremap <silent> * :call VisualSearch('f')<CR>
vnoremap <silent> # :call VisualSearch('b')<CR>

" 用c-j,k在buffer之间切换
nn <C-J> :bn<cr>
nn <C-K> :bp<cr>
nn <C-H> :b1<cr>

" ==================================================
"                快捷键运行脚本
" ==================================================
" <F5>运行python3脚本
map <F5> :call RunPython()<CR>
func! RunPython()
    exec "W"
    if &filetype == 'python'
        exec "!time python3 %"
    endif
endfunc
```

# 配置C/C++开发环境
1. 安装基础包
```shell
yay -S nodejs npm vim 
```
2. 如果使用makefile而不是cmake,还需要安装
```shell
yay -S bear
```
每次使用`bear make`生成配置文件
3. 安装插件管理，强烈推荐`vim-plug`
```shell
yay -S vim-plug
```
或
```shell
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```
4. 编写`.vimrc`
```vim
" 以下每个插件，都可使用https://github.com/***查看其官方介绍
call plug#begin('~/.vim/plugged')
Plug 'Yggdroot/indentLine'                          "显示对齐线
Plug 'scrooloose/nerdtree'                          "显示文件目录
Plug 'scrooloose/nerdcommenter'                     "用于快速高效注释代码
Plug 'neoclide/coc.nvim', {'branch': 'release'}     "基于nodejs的智能补全插件，非常强大
Plug 'vim-airline/vim-airline'                      "友好的显示状态栏
Plug 'peterhoeg/vim-qml'                            "qml高亮
Plug 'octol/vim-cpp-enhanced-highlight'             "cpp高亮
Plug 'ryanoasis/vim-devicons'                       "显示文件类型图标，系统需安装图标字体：yay -S nerd-fonts-complete
Plug 'junegunn/fzf.vim'                             "文件搜索工具，yay -S fzf the_silver_searcher
call plug#end()

set nocompatible    " 关闭兼容模式
syntax on       " 语法高亮
" 开启文件类型侦测
filetype on
" 根据侦测到的不同类型加载对应的插件
filetype plugin on  " 文件类型插件
filetype indent on
set autoindent
autocmd BufEnter * :syntax sync fromstart
" 设置背景透明
hi Normal ctermfg=252 ctermbg=none

set nu              " 显示行号
set showcmd         " 显示命令
set lz              " 当运行宏时，在命令执行完成之前，不重绘屏幕
set hid             " 可以在没有保存的情况下切换buffer
set backspace=eol,start,indent
set whichwrap+=<,>,h,l " 退格键和方向键可以换行
set incsearch       " 增量式搜索
set hlsearch        " 搜索时，高亮显示匹配结果。
" set ignorecase      " 搜索时忽略大小写
set magic           " 额，自己:h magic吧，一行很难解释
set showmatch       " 光标遇到圆括号、方括号、大括号时，自动高亮对应的另一个圆括号、方括号和大括号。
set nowb
set noswapfile      " 不使用swp文件，注意，错误退出后无法恢复
set lbr             " 在breakat字符处而不是最后一个字符处断行
set ai              " 自动缩进
set si              " 智能缩进
set cindent         " C/C++风格缩进

"命令模式下，底部操作指令按下 Tab 键自动补全。第一次按下 Tab，会显示所有匹配的操作指令的清单；第二次按下 Tab，会依次选择各个指令。
set wildmenu
set wildmode=longest:list,full

set nofen
set fdl=10
" tab转化为4个字符
set expandtab
set smarttab
set shiftwidth=4
set tabstop=4
" 不使用beep或flash
set vb t_vb=
"set background=dark
"colorscheme elflord
" 启用256色
set t_Co=256
set t_ut=n

set history=50  " vim记住的历史操作的数量，默认的是20
set autoread     " 当文件在外部被修改时，自动重新读取
set mouse=a      " 在所有模式下都允许使用鼠标，还可以是n,v,i,c等

" 光标所在的当前行高亮
set cursorline
" highlight CursorLine   cterm=NONE ctermbg=gray ctermfg=black guibg=NONE guifg=NONE
" highlight CocHighlightText cterm=NONE ctermbg=yellow ctermfg=black guibg=NONE guifg=NONE

" 高亮多余的空白字符及Tab
highlight RedundantSpaces ctermbg=red guibg=red
match RedundantSpaces /\s\+$\| \+\ze\t\|\t/

" 插件窗口的宽度，如TagList,NERD_tree等，自己设置
let s:PlugWinSize = 28

" 自动完成括号和引号
inoremap <leader>1 ()<esc>:let leavechar=")"<cr>i
inoremap <leader>2 []<esc>:let leavechar="]"<cr>i
inoremap <leader>3 {}<esc>:let leavechar="}"<cr>i
inoremap <leader>4 {<esc>o}<esc>:let leavechar="}"<cr>O
inoremap <leader>q ''<esc>:let leavechar="'"<cr>i
inoremap <leader>w ""<esc>:let leavechar='"'<cr>i

" 用 */# 向 前/后 搜索光标下的单词
vnoremap <silent> * :call VisualSearch('f')<CR>
vnoremap <silent> # :call VisualSearch('b')<CR>

" 用c-j,k在buffer之间切换
nn <C-J> :bn<cr>
nn <C-K> :bp<cr>
nn <C-H> :b1<cr>

let mapleader=','

"----------------------------------------------------
"                    indentLine
"----------------------------------------------------
let g:indentLine_enabled = 1
let g:indent_guides_guide_size            = 1  " 指定对齐线的尺寸
let g:indent_guides_start_level           = 2  " 从第二层开始可视化显示缩进

"----------------------------------------------------
"                    nerdtree
"----------------------------------------------------
" 进入 vim 时自动开启 NERDTree
autocmd VimEnter * NERDTree | wincmd p
" 若关闭某个 buff 后 NERDTree 是仅剩的最后一个 buff，则自动关闭 NERDTree
autocmd BufEnter * if tabpagenr('$') == 1 && winnr('$') == 1 && exists('b:NERDTree') && b:NERDTree.isTabTree() | quit | endif
" 使用PlugWinSize
let NERDTreeWinSize = s:PlugWinSize
" 使用 Ctrl+n 快捷键打开或关闭 NERDTree
nnoremap <C-n> :NERDTreeToggle<CR>


"----------------------------------------------------
"                    fzf
"----------------------------------------------------
"fzf快捷键 ctrl+p
nnoremap <silent> <C-p> :Files<CR>
nnoremap <silent> <C-k> :Buffers<CR>
nnoremap <silent> <C-a> :Ag<CR>
" 若需要设置过滤路径：
" yay -S fd
" vim ~/.zshrc
" export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow --exclude .git'
" vim ~/.config/fd/ignore
" 输入要过滤的文件或文件夹，如：
" build
" build-Release

"----------------------------------------------------
"                    coc.nvim
"----------------------------------------------------
" Set internal encoding of vim, not needed on neovim, since coc.nvim using some
" unicode characters in the file autoload/float.vim
set encoding=utf-8

" TextEdit might fail if hidden is not set.
set hidden

" Some servers have issues with backup files, see #649.
set nobackup
set nowritebackup

" Give more space for displaying messages.
set cmdheight=2

" Having longer updatetime (default is 4000 ms = 4 s) leads to noticeable
" delays and poor user experience.
set updatetime=300

" Don't pass messages to |ins-completion-menu|.
set shortmess+=c

" Always show the signcolumn, otherwise it would shift the text each time
" diagnostics appear/become resolved.
if has("nvim-0.5.0") || has("patch-8.1.1564")
  " Recently vim can merge signcolumn and number column into one
  set signcolumn=number
else
  set signcolumn=yes
endif

" Use tab for trigger completion with characters ahead and navigate.
" NOTE: Use command ':verbose imap <tab>' to make sure tab is not mapped by
" other plugin before putting this into your config.
inoremap <silent><expr> <TAB>
      \ pumvisible() ? "\<C-n>" :
      \ CheckBackspace() ? "\<TAB>" :
      \ coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"

function! CheckBackspace() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction

" Use <c-space> to trigger completion.
if has('nvim')
  inoremap <silent><expr> <c-space> coc#refresh()
else
  inoremap <silent><expr> <c-@> coc#refresh()
endif

" Make <CR> auto-select the first completion item and notify coc.nvim to
" format on enter, <cr> could be remapped by other vim plugin
inoremap <silent><expr> <cr> pumvisible() ? coc#_select_confirm()
                              \: "\<C-g>u\<CR>\<c-r>=coc#on_enter()\<CR>"

" Use `[g` and `]g` to navigate diagnostics
" Use `:CocDiagnostics` to get all diagnostics of current buffer in location list.
nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)

" GoTo code navigation.
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" Use K to show documentation in preview window.
nnoremap <silent> K :call ShowDocumentation()<CR>

function! ShowDocumentation()
  if CocAction('hasProvider', 'hover')
    call CocActionAsync('doHover')
  else
    call feedkeys('K', 'in')
  endif
endfunction

" Highlight the symbol and its references when holding the cursor.
autocmd CursorHold * silent call CocActionAsync('highlight')

" Symbol renaming.
nmap <leader>rn <Plug>(coc-rename)

" Formatting selected code.
xmap <leader>f  <Plug>(coc-format-selected)
nmap <leader>f  <Plug>(coc-format-selected)

augroup mygroup
  autocmd!
  " Setup formatexpr specified filetype(s).
  autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
  " Update signature help on jump placeholder.
  autocmd User CocJumpPlaceholder call CocActionAsync('showSignatureHelp')
augroup end

" Applying codeAction to the selected region.
" Example: `<leader>aap` for current paragraph
xmap <leader>a  <Plug>(coc-codeaction-selected)
nmap <leader>a  <Plug>(coc-codeaction-selected)

" Remap keys for applying codeAction to the current buffer.
nmap <leader>ac  <Plug>(coc-codeaction)
" Apply AutoFix to problem on the current line.
nmap <leader>qf  <Plug>(coc-fix-current)

" Run the Code Lens action on the current line.
nmap <leader>cl  <Plug>(coc-codelens-action)

" Map function and class text objects
" NOTE: Requires 'textDocument.documentSymbol' support from the language server.
xmap if <Plug>(coc-funcobj-i)
omap if <Plug>(coc-funcobj-i)
xmap af <Plug>(coc-funcobj-a)
omap af <Plug>(coc-funcobj-a)
xmap ic <Plug>(coc-classobj-i)
omap ic <Plug>(coc-classobj-i)
xmap ac <Plug>(coc-classobj-a)
omap ac <Plug>(coc-classobj-a)

" Remap <C-f> and <C-b> for scroll float windows/popups.
if has('nvim-0.4.0') || has('patch-8.2.0750')
  nnoremap <silent><nowait><expr> <C-f> coc#float#has_scroll() ? coc#float#scroll(1) : "\<C-f>"
  nnoremap <silent><nowait><expr> <C-b> coc#float#has_scroll() ? coc#float#scroll(0) : "\<C-b>"
  inoremap <silent><nowait><expr> <C-f> coc#float#has_scroll() ? "\<c-r>=coc#float#scroll(1)\<cr>" : "\<Right>"
  inoremap <silent><nowait><expr> <C-b> coc#float#has_scroll() ? "\<c-r>=coc#float#scroll(0)\<cr>" : "\<Left>"
  vnoremap <silent><nowait><expr> <C-f> coc#float#has_scroll() ? coc#float#scroll(1) : "\<C-f>"
  vnoremap <silent><nowait><expr> <C-b> coc#float#has_scroll() ? coc#float#scroll(0) : "\<C-b>"
endif

" Use CTRL-S for selections ranges.
" Requires 'textDocument/selectionRange' support of language server.
nmap <silent> <C-s> <Plug>(coc-range-select)
xmap <silent> <C-s> <Plug>(coc-range-select)

" Add `:Format` command to format current buffer.
command! -nargs=0 Format :call CocActionAsync('format')

" Add `:Fold` command to fold current buffer.
command! -nargs=? Fold :call     CocAction('fold', <f-args>)

" Add `:OR` command for organize imports of the current buffer.
command! -nargs=0 OR   :call     CocActionAsync('runCommand', 'editor.action.organizeImport')

" Add (Neo)Vim's native statusline support.
" NOTE: Please see `:h coc-status` for integrations with external plugins that
" provide custom statusline: lightline.vim, vim-airline.
set statusline^=%{coc#status()}%{get(b:,'coc_current_function','')}

" Mappings for CoCList
" Show all diagnostics.
nnoremap <silent><nowait> <space>a  :<C-u>CocList diagnostics<cr>
" Manage extensions.
nnoremap <silent><nowait> <space>e  :<C-u>CocList extensions<cr>
" Show commands.
nnoremap <silent><nowait> <space>c  :<C-u>CocList commands<cr>
" Find symbol of current document.
nnoremap <silent><nowait> <space>o  :<C-u>CocList outline<cr>
" Search workspace symbols.
nnoremap <silent><nowait> <space>s  :<C-u>CocList -I symbols<cr>
" Do default action for next item.
nnoremap <silent><nowait> <space>j  :<C-u>CocNext<CR>
" Do default action for previous item.
nnoremap <silent><nowait> <space>k  :<C-u>CocPrev<CR>
" Resume latest coc list.
nnoremap <silent><nowait> <space>p  :<C-u>CocListResume<CR>

"----------------------------------------------------
"               vim-cpp-enhanced-highlight
"----------------------------------------------------
" 默认情况下禁用类范围的突出显示。 启用设置
let g:cpp_class_scope_highlight = 1
" 默认情况下禁用成员变量的突出显示。 启用设置 
let g:cpp_member_variable_highlight = 1
" 默认情况下，声明中类名的突出显示是禁用的。 启用设置 
let g:cpp_class_decl_highlight = 1
" 默认情况下禁用 POSIX 函数的突出显示。 启用设置
let g:cpp_posix_standard = 1
" 可以突出显示模板功能
let g:cpp_experimental_template_highlight = 1
" 启用库概念的突出显示
let g:cpp_concepts_highlight = 1
```
编写完成后保存，然后执行：
```vim
:PlugInstall
```
等待插件安装完成，安装coc插件：
```vim
:CocInstall coc-json coc-clangd
```
装完之后使用方式，首先cd到项目目录：
```shell
mkdir build
cd build
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=1
```
然后使用vim打开项目文件即可正常编辑，可以愉快的享受智能提示及语法高亮了！

