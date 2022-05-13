.PHONY: build
default: run


args = `arg="$(filter-out $@,$(MAKECMDGOALS))" && echo $${arg:-${1}}`

test :
		@echo $(call args,defaultstring)
run:
		hexo server
deploy:
		hexo clean && hexo deploy
new:
		hexo new $(call args,defaultstring)
draft:
		hexo new draft $(call args,defaultstring)
publish:
		hexo  publish $(call args,defaultstring)
