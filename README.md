(add-to-list 'package-archives
	     '("marmalade" . "http://marmalade-repo.org/packages/"))
(prelude-require-packages '(company irony company-irony company-c-headers))

(add-to-list 'c++-mode-hook (lambda () (c-set-offset 'innamespace 0)))

(require 'company)
(add-hook 'after-init-hook 'global-company-mode)
(require 'irony)
(add-hook 'c++-mode-hook 'irony-mode)
(add-hook 'c-mode-hook 'irony-mode)
(add-hook 'objc-mode-hook 'irony-mode)
(define-key company-active-map (kbd "<tab>") (lambda () (interactive) (company-complete-common-or-cycle 1)))
(define-key company-active-map (kbd "<backtab>") (lambda () (interactive) (company-complete-common-or-cycle -1)))

;; replace the `completion-at-point' and `complete-symbol' bindings in
;; irony-mode's buffers by irony-mode's function
(defun my-irony-mode-hook ()
  (define-key irony-mode-map [remap completion-at-point]
    'irony-completion-at-point-async)
  (define-key irony-mode-map [remap complete-symbol]
    'irony-completion-at-point-async))
(add-hook 'irony-mode-hook 'my-irony-mode-hook)
(add-hook 'irony-mode-hook 'irony-cdb-autosetup-compile-options)

(eval-after-load 'company
  '(add-to-list 'company-backends 'company-irony))
(add-hook 'irony-mode-hook 'company-irony-setup-begin-commands)
(eval-after-load 'company
  '(add-to-list 'company-backends 'company-c-headers))


(defun smart-tab ()
  “This smart tab is minibuffer compliant: it acts as usual in
   the minibuffer. Else, if mark is active, indents region. Else if
   point is at the end of a symbol, expands it. Else indents the
   current line.”
   (interactive)
   (if (minibufferp)
       (unless (minibuffer-complete)
         (dabbrev-expand nil))
     (if mark-active
         (indent-region (region-beginning)
                        (region-end))
       (if (looking-at “\\_>”)
           (dabbrev-expand nil)
         (indent-for-tab-command)))))


(setq prelude-whitespace nil)

;;
;; Linux kernel coding style
;;
(defun c-lineup (ignored)
  "Line up argument lists by tabs"
  (let* ((anchor (c-langelem-pos c-syntactic-element))
         (column (c-langelem-2nd-pos c-syntactic-element))
         (offset (- (1+ columg) anchor))
         (steps (floor offset c-basic-offset)))
    (*(max steps 1)
      c-basic-offset)))

(add-hook 'c-mode-common-hook
          (lambda()
            (c-add-style
             "linux-tabs-only"
             '("linux" (c-offsets-alist
                        (arglist-cont-nonempty
                         c-lineup-gcc-asm-reg
                         c-lineup))))))
(add-hook 'c-mode-hook
          (lambda()
            (setq indent-tabs-mode t)
            (setq show-trailing-whitespace t)
            (c-set-style "linux-tabs-only")))



(setq-default indent-tabs-mode t)
(setq-default tab-width 8)
(setq-default tab-always-indent 'complete)

;;
;; Bracers and stuff
;;
(require 'cc-mode)
(defun my-c-mode-insert-lcurly ()
  (interactive)
  (insert "{")
  (let ((pps (syntax-ppss)))
    (when (and (eolp) (not (or (nth 3 pps) (nth 4 pps)))) ;; EOL and not in string or comment
      (c-indent-line)
      (insert "\n\n}")
      (c-indent-line)
      (forward-line -1)
      (c-indent-line))))
(define-key c-mode-base-map "{" 'my-c-mode-insert-lcurly)

(prelude-require-package 'column-enforce-mode)
(add-hook 'c-mode-hook 'column-enforce-mode)
;; ;;
;; ;; Switch between header and implementationfile with C-c h h
;; ;;
;; (add-hook 'c-mode-common-hook
;; 	  (lambda()
;; 	    local-set-key (kbd "C-c h") 'ff-find-other-file))

;;
;; Yasnippet
;;;
(prelude-require-package 'yasnippet)
(add-hook 'c-mode-hook
	  '(lambda ()
	     (yas-minor-mode)))
(yas-global-mode 1)

;;
;; Remap a couple of keys to make coding on a MacBook easier
;;
(global-set-key (kbd "s-(") "[")
(global-set-key (kbd "s-)") "]")
(global-set-key (kbd "s-4") "$")
(global-set-key (kbd "s-8") "{")
(global-set-key (kbd "s-9") "}")
(global-set-key (kbd "M-/") "\\")
(global-set-key (kbd "M-7") "|")
(global-set-key (kbd "M-2") "@")

(global-linum-mode 1)



;;------------------------------------------
;; Scons
;;------------------------------------------
;; Tells Emacs to use scons instead of make to compile
(setq compile-command "scons -Q")

;; Load python-mode for scons files
(setq auto-mode-alist(cons '("SConstruct" . python-mode) auto-mode-alist))
(setq auto-mode-alist(cons '("SConscript" . python-mode) auto-mode-alist))
